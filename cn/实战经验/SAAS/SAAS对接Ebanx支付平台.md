# SAAS对接Ebanx支付平台

主要用于BOOLV项目在巴西市场介入Ebanx支付网关，实现pix-auto（自动pix扣款）的订阅产品，用户可以通过扫码变更强或者链接来完成首次支付并建立自动扣费。 后端主要为了处理状态同步，权益开通和自动扣费环节。



## 1.业务

### 1. 用户发起订阅

当用户在前端点击订阅的按钮的时候，前端会给我们后端的API 传入订阅的参数(userid,价格，订阅周期，套餐等)

这时候后端就要去：

1.做是否重复订阅的检查

| 情况           | 用户状态            | 我的处理               | 原因                             |
| -------------- | ------------------- | ---------------------- | -------------------------------- |
| **活跃订阅**   | 已支付，有权益      | ❌ 拒绝创建新订阅       | 他已经买了，不该再扣一次钱       |
| **待处理订阅** | 生成了 QR，还没支付 | ✅ 返回上次的 QR 和链接 | 没必要生成新的，让他继续付上次的 |
| **无订阅记录** | 全新用户/上次已取消 | ✅ 创建新订阅           | 正常流程                         |

如果无订阅记录那么就创建唯一标识

然后**生成唯一标识**：

- [merchantPaymentCode](vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) = "pay-{appId}-{userId}-{timestamp}" —— 用来标识"这笔付款"
- [merchantEnrollmentCode](vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) = "enr-{appId}-{userId}-{timestamp}" —— 用来标识"这个签约"（用于后续自动扣款）

**这两个 ID 的作用**：

- merchantPaymentCode 用来追踪**首次付款**的状态（成功/失败/取消）
- merchantEnrollmentCode 用来追踪整个**订阅协议**的状态，以及后续每个月的自动扣款都会关联到这个 enrollment

### 2. 向Ebanx发起 支付 + 签约请求

后端会构造一个http请求给Ebanx的/ws/direct接口 携带3个信息

~~~cmd
付款信息（payment）

merchantPaymentCode（刚才生成的）
amountTotal（首月金额，比如 ¥99）
currencyCode = BRL（巴西雷亚尔）
email/name/document（用户信息）
expirationTimeInSeconds（这个 QR 码多久过期，通常 1 天）
签约信息（enrollment）

merchantEnrollmentCode（刚才生成的）
同样的 email/name/document
country = BR
订阅信息（subscription）

subscriptionName（比如 "Premium Monthly"）
frequency = MONTHLY（月度续费）
fixedAmount（每月金额）
startDate（下一个周期的开始日期，我会用 FrequencyUtils 计算好）
retryPolicy = ALLOW_3R_7D（失败后最多重试 3 次，间隔 7 天）
~~~

### 3.Ebanx返回QR码和链接

EBANX 的响应会给我：

~~~cmd
- payment.hash —— 这很重要，后续查询这笔支付的状态需要用 hash 而不是 code（因为 code 可能被伪造）
- payment.pix_automatico.qrCodeValue —— 用户扫这个二维码就能支付
- redirectUrl —— 也可以通过这个链接直接跳转到支付页面
~~~

然后把这些信息存储到mysql

~~~cmd
我会把以下信息存进数据库 ebanx_subscription 表：

userId / appId / enrollmentCode = merchantEnrollmentCode
enrollmentStatus = PENDING（表示还在等待用户完成支付）
currentPeriodEnd（根据 frequency 计算的本周期结束时间）
price（金额）
frequency（周期类型）
metadata（JSON 格式存放 QR 码、redirect URL、用户信息等）
同时在 ebanx_payment 表记录：

merchantPaymentCode
hash（来自 EBANX 响应）
state = PE（pending，待支付）
amount / currency
enrollmentCode
~~~

### 4. 用户扫码付米

用户在pix端付款了以后，ebanx收到钱然后我们收到ebanx的回调，进行更新状态

~~~cmd
payment.status 从 PE 变成 CO（completed，已到账）
enrollment.status 从 PENDING 变成 ACCEPTED（表示用户已确认并完成支付，签约生效）
~~~

### 5,状态同步 （两条路并行）

EBANX 发送 webhook 到我的 [/ebanx/webhook](vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) 端点，通知状态变更。

**当前实现**：我接收 webhook 不直接信任内容（避免被伪造 webhook 坑），而是作为一个"触发信号"，然后立即调用后面讲的状态查询逻辑。 （我不信任是因为Ebanx是我见过最烂的第三方，他只会告诉我用户状态变更了，但是又不告诉我变更为什么）

即使 webhook 丢失或延迟，我的定时任务（XXL-Job）会每隔一段时间（比如 5 分钟）自动检查所有 PENDING 的订阅，调用 EBANX 的 [/ws/userenrollments/query](vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) 接口查询最新状态。

这么做主要

~~~cmd
webhook 快速响应（用户秒级感知到订阅成功）
定时任务保兜底（万一 webhook 没到，也能在下个轮询周期捞起）
即使同时触发也没关系，因为我用了 Redis 分布式锁 来保证同一用户的更新不会冲突
~~~

### 6. 确认状态确认支付成功(核心环节)

无论通过哪条路触发，最终都会执行这个逻辑

**第一次查询：enrollment 状态**

- 调用 [/ws/userenrollments/query](vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)，用 merchantEnrollmentCode 查询
- 得到 enrollment.status：如果是 ACCEPTED，表示用户已确认

第二次查询 payment 确保钱已经到我公司上了

- 拿出刚才保存的 [payment.hash](vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)，调用 [/ws/query](vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) 查询
- 确认 payment.status == CO（已到账）

这两个状态都确认后，我才敢开通权益。

### 7.更新本地数据库

- [ebanx_subscription.enrollmentStatus = ACCEPTED](vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)（标记签约生效）
- [ebanx_payment.state = CO](vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)（标记付款已完成）
- 记录 `notified_at = now（）`（用于追踪什么时候开通的）

然后调用我们主系统进行发送权益。

#### 这时候有一些幂等性保护

开发送权益之前我们回检测[ebanx_subscription.enrollmentStatus  是不是 ACCEPTED，如果是说明成功开通了，直接返回200，不要再重复开通。

同时我们的主业务也会在做一次防御，会根据我们的subcode进行判断是否已经开通，这样我们webhook重复投递或者定时任务重复执行，用户也会被重复开通会这重复扣费。

#### 并发控制

使用Redisson分布式锁， 

- `lock = redissonClient.getLock("ebanx-subscribe-lock-" + userId)`
- 在进行任何更新操作之前获取锁，确保同一个用户的订阅操作全局串行化
- 即使同时有 webhook 和定时任务都在处理同一用户的状态变更，也只有一个会实际更新

#### **事务保护**：

任何一个地方出错直接回滚

### 8.自动扣费

订阅开通后，每个月系统会自动续费。续费流程类似，但更简单：

**检测即将续费的订阅**

- 定时任务查找所有 [enrollmentStatus == ACCEPTED](vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) 且 [currentPeriodEnd < now + 3 days](vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) 的订阅
- 提前 3 天预检，给续费留出充足的时间窗口 （3天是根据巴西法律）

**发起周期扣款**

- 调用 `/ws/direct`（同样的端点），但这次 operation = "request"（表示这是一个周期性扣款而不是首次）
- 新建一个 merchantPaymentCode，并关联到已有的 merchantEnrollmentCode
- 关键细节：dueDate = max(currentPeriodEnd, now + 2 days)
  - 这是为了处理"定时任务丢失"的极端情况
  - 如果任务延迟触发，dueDate 可能会小于现在的时间，EBANX 会拒绝
  - 所以我们采用较大值，确保时间总是向前的

**本地记录**

- 新建 [EbanxPaymentEntity](vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) 记录（state = PE）
- 更新 subscription（currentPeriodStart = 旧的 currentPeriodEnd, currentPeriodEnd = nextPeriod, enrollmentStatus = PENDING）
- 标记最新的支付 ID 关联上



“subCode” 是我们主业务里用来表示“外部订阅/签约/订阅 ID”的字段（也就是把第三方支付/订阅服务的订阅 id 映射到我们系统里的唯一标识）。对于 EBANX，我们把 [merchantEnrollmentCode](vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) 当作 [subCode](vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) 传入主业务，用它来做幂等/判重与订阅记录关联，防止重复开通与重复扣费。

**等待确认**

- 之后的 webhook/轮询 -> 查询 -> 开通流程和首次订阅一样
- 确认成功后，用户在下一个月期间仍然可以使用产品

### 9取消订阅或退款

如果用户想取消或出现异常情况：

- 调用 `/ws/userenrollment`（cancel 操作）取消签约
- 调用 [/ws/refundOrCancel](vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) 处理退款
- 本地标记 [enrollmentStatus = REVOKED](vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html)
- 调用主业务 [cancelSubImmediately()](vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) 立即冻结用户权益

## 扩展问题

### 1. 并发重复订阅

**问题**：用户快速点两次订阅按钮，可能导致两条重复记录。
**方案**：用 Redisson 锁 + DB 检查。每次创建前先查 PENDING 记录，如果有直接返回已有的 QR 码。

### 2. 状态不一致

**问题**：webhook 丢失、网络抖动等导致本地状态和 EBANX 状态不同步。
**方案**：定时任务轮询所有 PENDING 的订阅，主动查询 EBANX 状态并同步。双保险。

### 3. 重复开通权益

**问题**：webhook 重复投递，可能导致用户被重复开通或重复扣费。
**方案**：开通前做幂等检查（先看本地是否已 ACCEPTED），主业务层也幂等。

### 4. 定时任务延迟导致续费失败

**问题**：如果定时任务延迟，计算出来的 dueDate 可能小于当前时间，EBANX 会拒绝。
**方案**：用 [dueDate = max(currentPeriodEnd, now + 2 days)](vscode-file://vscode-app/Applications/Visual Studio Code.app/Contents/Resources/app/out/vs/code/electron-browser/workbench/workbench.html) 总是确保向前。

### 5. 跨实例并发更新

**问题**：多个 App Server 实例同时处理同一用户的状态变更。
**方案**：Redis 分布式锁（key: "ebanx-subscribe-lock-{userId}"），全局串行化。
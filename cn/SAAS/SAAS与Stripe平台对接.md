# SAAS与Stripe平台对接

## 1. 业务

我们 SAAS提供按照订阅扣费服务，支付和结算通过Stripe处理，SAAS系统只负责订阅生命周期的管理，还有权益发送和扣减，以及基于Stripe的webhook做事件驱动的财务和权限变更。

### 1. 支付流程

首先客户端请求购买，这时候后端就娇艳商品的合法性还有用户还有计算应受金额和优惠和有效期。

一切通过后端会创建一个stripe支付回话

​	首先会创建一个订阅checkout session （model=subscription），或者先创建Customer在调用Subscription APi。

​	然后后端生成保存本次请求的幂等性Key（order_id）,然后把它存在Orders表，

​	然后返回给用户端client_secret(PaymentIntent)或者session id

~~~java
具体返回的是client_secret(PaymentIntent)或者session id取决于

想省时间且合规简单：使用 Checkout（返回 session_id），把复杂的 SCA 与 UI 都由 Stripe 托管。
想自定义体验并在前端处理 3DS：使用 Subscriptions API，返回 PaymentIntent.client_secret 给前端去完成认证。
~~~

​	然后用户完成支付我们后端等待webhook的回调，常见的事件有

```java
checkout.session.completed - 支付完成
payment_intent.succeeded - 支付成功
invoice.paid - 支付成功
invoice.payment_failed - 支付失败
charge.refunded - 已退款
```

当我们后端收到了回调以后首先会校验Stripe- signature（使用endpoint secret（密钥））冰验证timestamp防止replay。

然后解析出来event 获得event.id event.type,envent.object(包含payment_intent,invoice,subscription)

然后检查幂等，见擦好我们的webhookEvents表是否存在stripe_evnet_id，如果已经存在了正确其他的worker在处理了，直接返回200。

然后创建和更新的时候记得在表里维护好状态。比如

~~~java
创建或更新 Transactions：字段包含 tx_id（内部）、stripe_event_id、stripe_payment_intent_id、amount、currency、status。
~~~

然后把发送的工作交给异步消息队列进行发送权益。

首先检查这个事件的权益是否发送，如果没发生就执行发送逻辑，发送成功email通知用户。



## 2. 扩展问题

### 1.幂等性、并发与原子

Webhook幂等：webhookEvents表的stripe_event_id设置唯一索引，处理前先尝试插入，若已存在则根据状态判断是否跳过。

关于是否重复发送会在表里根据一个字段判断是否重复发送，

所有权益扣减都是通过reqId然后通过redis分布式锁进行校验是否多扣

### 2. 故障场景和补偿策略

webhook多次发送，幂等策略避免重复发送

尝试多次发送失败的放到死信队列里由人工处理。然后还有一个定时任务，定期尝试发送权益。

如果退款收到回调就计算未使用的比例，然后执行回收或者人工流产。然后标记状态

争议就人工处理。

### **3. stripe Checkout vs PaymentIntent vs SetupIntent —— 各用在什么场景？**

- Checkout（mode=payment/subscription）：托管页面，简单快速，SCA/3DS 由 Stripe 处理，推荐用于标准场景。
- PaymentIntent：自定义 UI + 前端 3DS 确认，更灵活但开发复杂。
- SetupIntent：只保存卡不扣款，用于 trial 或先存卡后续自动扣。
- 订阅首付通常用 Checkout；自定义体验用 PaymentIntent.client_secret；trial 阶段用 SetupIntent。

### **4. 订阅时什么时候返回 session_id，什么时候返回 PaymentIntent.client_secret？**

- Checkout 路径（托管）→ 返回 session_id，前端 redirect。
- 自定义 + 需前端确认 3DS → 返回 PaymentIntent.client_secret，前端调用 confirmCardPayment()。
- 关键：后端最终都需依赖 webhook（invoice.paid）作最终确认，不要只信前端成功回调。

### 5. 前端收到了支付成功，但是webhook没收到怎么办

不要相信前端的任何数据，一切都等webhook回调。或者后端主动查询状态。

### 6.**如何保证 PCI 合规？为什么不能自己存储卡号？**

- ​	 使用 Stripe Elements / Checkout，前端直接把卡信息送到 Stripe（不过你的服务器）。
  - 后端只处理 Stripe 返回的 token / payment_method id（非敏感数据）。
  - 这样符合 PCI DSS SAQ-A（最低级别责任）。
  - 如果自己存卡号，必须是 PCI-DSS L1 认证（很贵）。

### 7.**如何验证 webhook 确实来自 Stripe？**

验证stripe头的stripe-signature,用endpoint secret 计算出来校验。

然后验证timestamp 可以有5分钟的容忍度，防止replay攻击

### 8. 怎么保证不会重复发送权益，

根据stripe_event_id 设置唯一索引，保证这个交易只能发一次。

### 9. 如果乱序到达怎么半 

比如先收到了**先收到 invoice.paid 再收到 payment_intent.succeeded**

理论上不会有任何问题，因为有event id，不管谁先来了都是发一次。

但是我们业务代码必须要求按照顺序，可以先把事件入库，然后通过单线程worker处理（必须保证顺序）

### 10.**同时多个 worker 处理同一条 transaction，怎么保证只发放一次？**

通过event_id 设置唯一索引。

### 11. **webhook 处理失败（如 DB down），应该怎么办**

立即报错500stripe会重试，如果失败n次了，可以扔到死信队列DLQ，或者本地DB。

然后立刻警报。然后在重试发送。定期扫描未完成的transaction（权益还没发的）进行补发

### 12. **支付了但权益没发放，怎么做对账？**

 从stripe拉财务数据，然后和内部账务做比对

- Stripe 有但本地没 → 补发权益或告警。
- 本地有但 Stripe 没 → 可能是还没结算，mark for manual review。

### 13. **用户发起退款，权益应该如何回收？**

- 收到 `charge.refunded` 或 `invoice.refund` webhook。
- 根据产品政策决定：
  - **不回收已消费的权益**（如已用的点数）。
  - **按比例回收未使用**（如剩余月度额度）。
  - 或**记录欠款**，下次扣款时调整。
- 创建补偿性 transaction（反向记录）。
- 通知用户退款理由。
- 若是争议（dispute），先冻结权益，等人工审核结果。

### 14. **订阅从月付升级到年付，如何计费？**

- 使用 Stripe 的 **proration** 功能：Stripe 会自动计算差价。然后清旧的权益发新的

### 15 **如何设计高并发场景下的权益发放系统？**

- - **事件驱动 + 异步**：webhook 控制器 快速入队（100ms），worker 后续处理。
  - **队列隔离**：使用 RabbitMQ，支持多个 worker 并行处理，支持重试和 DLQ。
  - **幂等保证**：DB 唯一索引、事务、version control（乐观锁）。
  - **缓存加速**：Redis 存用户当前额度，异步持久化到主 DB，定期对账。
  - **限流与背压**：若发放队列堆积，暂停接收新 webhook（或返回 429 让 Stripe 重试）。
  - **监控 & alerting**：关键指标（成功率、延迟、队列深度）。

###  **16 如何保证最终一致性（支付确认 <-> 权益发放）？**

- Step 1: webhook 记录（可信来源）。
- Step 2: 发放权益（可重试）。
- Step 3: 标记发放完成。

- 若 Step 2 失败，通过补偿任务重试（直到成功）。
- **对账**：定期 reconciliation task 检查是否一致，不一致则重发。
- **关键**：保持 event log（WebhookEvents）作单一真实来源，其他操作围绕它重试。

### 18.  **如何处理 Stripe 与本地数据不一致？**

- **信任 Stripe 是单一真实来源**（财务数据）。
- 定期对账任务找差异。
- 修复策略：
  - 若本地多了 transaction → 标记为需调查（可能是测试数据）。
  - 若 Stripe 多了 → 补发权益。
  - 若金额不一致 → 记录差异并告警，由财务/法务审核。
- 记录所有对账结果供审计。

###  19. **Webhook retry 的指数退避如何配置？**

- A 要点：
  - Stripe 会重试 3 次（默认），时间间隔：5m, 30m, 2h。
  - 你的 worker 若用消息队列（Kafka/RabbitMQ），也应该配置重试：
    - 第 1 次：立即重试。
    - 第 2 次：延迟 1 分钟。
    - 第 3 次：延迟 5 分钟。
    - 第 4 次：进入 DLQ，告警。
  - 指数退避公式：`delay = base_delay * (2 ^ attempt_count)`。

### **20. 如何设计幂等 key（idempotency key）？**

- A 要点：
  - 格式：`{user_id}_{product_id}_{timestamp}_v{version}`。
  - 或使用 UUID + 某个业务标识。
  - 规则：同一个 idempotency key 在短时间内（如 24 小时）若重复请求，返回之前的结果。
  - 存储：在 DB 的 `Requests` 表，记录 key + response。
  - 使用场景：防止用户在前端重复点击"支付"按钮。

### 21. 如何确定一致性

所有这类的问题都是等webhook回调，和幂等处理

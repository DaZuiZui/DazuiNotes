# 视频上传Tiktok shop和鉴权

## 1.上传tiktok shop

### 1. 商品列表查询

获取Tiktok shop商品列表，用于关联视频

### 2.预检查

如果还没上传视频那么就上传视频到tiktok获取fileId ，上传过了直接返回id。

调用tiktok检测借口查看视频是否满足规则。

保存预检查内容到数据库

### 3.预检查状态

不断轮训获取检查结果（Checking/passed/failed）

返回检测结果

### 4. 发布视频

验证预检测是否通过

调用tiktok发布api

保存发布id和状态

### 5 发布状态查询

轮训检查是否已经成功发布

## 2.鉴权

鉴权的角色有2种

- CREATOR要求预先登录（因为是给现有用户增加功能）
- SELLER允许自动注册（因为是从商家后台导流，简化流程）

### 2.1 创作者creator

~~~java
1. 用户先在系统注册 ✅
   ↓
2. 登录BV系统 ✅
   ↓ (此时已有userId和token)
3. 在BV系统内点击"绑定TikTok"
   ↓
4. 跳转TikTok授权
   ↓
5. 回调到 /tiktok/user/login
   ↓
6. 检测到CREATOR类型
   ↓
7. StpUserService.currentUser() → 获取当前登录用户 ✅
   ↓
8. 将TikTok绑定到已有的BV账号
~~~

#### 流程

### 2.2 卖家

~~~java
1. 用户从TikTok点进来 
   ↓
2. 还没有本系统账号 
   ↓
3. 也没有登录本系统系统 
   ↓
4. 直接跳转TikTok授权
   ↓
5. 回调到 /tiktok/user/login
   ↓
6. 检测到SELLER类型
   ↓
7. StpUserService.currentUser() → 返回null 
   ↓
8. 自动创建BV账号 → 生成userId
  		用open_id tiktok唯一的标识
   ↓
9. 自动登录 → 生成token
   ↓
10. 将TikTok绑定到新创建的账号
~~~



### 1.授权登入

用TikTok OAuth回调返回的授权码然后获取到 用户名，邮箱，系统登入token，和isLogin是否登入，还有头像。


# Mysql 执行Select语句会发生的事情

## 连接器

第一步我们肯定是先建立连接才能进行执行sql语句。

Mysql也是TCP协议进行的，也需要3次挥手和4次挥手，当TCP建立连接后我们的连接器就会验证我们的用户名和密码，如果密码没有问题就会获取我们用户的权限，然后储存起来，然后这个用户所有操作都会根据权限进行一次逻辑判断看是否有权限（如果管理员中途修改了全县逻辑，那么也不会立刻生效只有下次连接时候才会生效。）

> 空闲连接

如果我们的用户连接完一直没有执行过命令也就是空闲的连接，空闲连接的最大等待时间是8小时，如果超过这个时间我们的mysql就会把他断开。



当然我们也可以通过以下命令进行手动断开。

~~~sql
kill connect id
~~~

> 最大连接数量

在我们的mysql默认的最大连接数量是151个。

### mysql解决长链接占用内存的问题

mysql执行查询过程中临时使用内存管理连接对象，这些连接对象资源只有在断开的时候才会释放，如果长连接积累太多，Mysql服务占用内存就会太大，就可能会被系统强制kill，导致mysql服务异常重启。

> 解决方案-客户端主动重制连接

在Mysql5.7版本实现了mysql_reset_connection()函数的接口，当我们客户端执行了一个很大的操作后，在代码里调用这个函数进行重制连接达到内存释放的效果，这个过程不需要重新连接和重新做权限验证，但是会把连接恢复到刚创建完成的状态。

## 缓存查询

​	这个功能就是以k-v形式进行缓存查询，如果缓存没有再去执行sql，key是我们的sql语句，value是我们的结果，但是表被更新了，那么这个表的所有缓存都失效了， 在Mysql8版本以后就删除了缓存这个机制。

## 解析器

### 词义分析

在select username from user会解析出来4个token其中关键字有select 和 from

**Token1:** select

**Token2:** username

**Token3:** from

**Token4:** user

如果是select username, password from user

**Token 1:** 类型是 `Keyword`，内容是 "select"。

**Token 2:** 类型是 `Identifier`，内容是 "username"。

**Token 3:** 类型是 `Symbol`，内容是 ","（逗号）。

**Token 4:** 类型是 `Identifier`，内容是 "password"。

**Token 5:** 类型是 `Keyword`，内容是 "from"。

**Token 6:** 类型是 `Identifier`，内容是 "userinfo"。

### 语法分析

语法分析就是判断我们的sql语句是否满足我们的mysql语法规则，如果满足就会创建对应的语法树，方便后面的板块获取sql类型、表名、字段名字、where条件。

> 在mysql5和8只负责解析语法和构建语法树，不负责查看字段和表是否存在.

## SQL执行

执行sql主要有3个阶段 prepare、optimize、execute。

**prepare：**预处理阶段。

**optimize：** 优化阶段。

**execute：** 执行阶段。

### 预处理器

检查我们sql语句的字段和表是否存在。

如果我们使用了select \*，那么就把\* 变成所有的字段.

### 优化器

这个阶段就是为我们的sql制定一个执行计划，比如我们表中有多个索引的时候，我们优化器会根据成本来考虑查使用哪一个索引。

比如我们使用这条sql，很明显会使用逐渐索引

~~~sql
select * from user where id = 1
~~~

比我们我们name也是普通索引，id是主键，我们的优化器就会考虑使用哪个成本最小，很明显使用name的索引最小，使用id的成本会大很多，因为二级索引就可以直接查询到结果。

```sql
select id from product where id > 1  and name like 'i%';
```

### 执行器

这个步骤执行我们的sq语句，从存储引擎拿到结果返回给客户端。
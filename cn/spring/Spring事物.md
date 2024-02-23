# Spring事物

​	Spring事物是Spring框架提供的一种机制，用于管理数据库事物和提交和回滚操作，事物是一组数据操作的执行单元，要不全部提交要不全部回滚，保证数据的一致性和完整性。

​	Spring提供了对事物的抽象和管理，使的开的这可以声明试的管理事务，而不需要手动编写繁琐的事务管理代码

## 事务的特性

### 	原子性 Atomicity

原子性代表着该事务要不全部成功要不全部失败会滚到初始状态。

### 	一致性 Consistency

执行业务的前后，数据保持一致。

​	举一个例子如果当前商品X有5个，事务A和事务B都要买商品X，事务A先执行让商品减去1个得到库存为4个，接下来事务B在执行，一样减去1个得到3个。如果当事务A读到商品是5个，B也是5个，他们减去1事务A变成了4个，B也是4个，就没有保证了一致性。当违反数据库的完整性约束或业务规则的时候，业务就会回滚。

### 	 隔离型 Isolation

并发访问数据库的时候，一个用户的事物不会被其他事物所干扰。各并发事物之间数据库是独立的。

### 	持久性 Durability

一旦一个事物被提交了，它对数据库中的数据改变是持久的。

## Spring常见的事物管理

### 编程式事物

使用transactionTemplate和TransactionManager手动管理事务	

~~~java
@Autowired
private TransactionTemplate transactionTemplate;
public void testTransaction() {

        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {

                try {
 
                } catch (Exception e){
               
                    transactionStatus.setRollbackOnly();
                }

            }
        });
}

~~~

### 声明试事务

@Transactional，使用的是AOP实现的。

## Spring 事务核心接口

### PlatformTransactionManager Spring事务管理核心接口

​	定义了事务管理器的基本功能，它包含了启动事物、提交事务、回滚事务等方法，用于管理事务的生命周期。

### TransactionDefinition 定义了事务的属性

​	该接口定义了事务的属性，包括隔离级别、传播行为、超时设置等。

​	**超时：**如果超过时间限制没有完成任务那就就自动回滚

### TransactionStatus 事务的状态

​	可以通过TransactionStatus获取事务的状态，如判断事物是为否为新事物、当前事物是否有保存点。

### TransactionOperations 事物以编程方式的执行的操作

​	通过编程方式控制事物的生命周期（开始和结束）和执行（事务的开始操作管理）。

### TransactionSynchronization	不同阶段的触发的回调方法

比如在事物的开始提交回滚定义一些回调方法执行自定义的逻辑业务。

## 回滚

只有在RuntimeException和Error才会导致我们事务回滚，但是遇见Checked异常的时候不会回滚。

## 事务的隔离级别

### READ UNCOMMITTED(读取未提交的数据)

最低的隔离级别，可以让事务读取未提交的数据，可能导致幻读、脏读、不可重复读的问题。

​	**脏读：**读到了另外一个事物没有提交的数据

​	**不可重复读：** 比如事物A第一个读取一条数据结果是1，事务A在读的时候这个值可能是2了，因为事务B对这个记录进行了修改。

### READ COMMITTED（读取已经提交的数据）

​	保证事务读取到了已经提交的数据，解决了脏读的问题，但是还是有不可以重复读和幻读。

​	**幻读：**事务A查询了某个表中某个范围内的记录，然后事务B插入了一条新记录，该记录也在事务A的查询范围内。如果事务A重新执行相同的查询，它会看到新插入的记录，这就是幻读问题

### REPEATEBLE READ (可重复读)

​	就是事物开启后所读取到的数据讲保持一致性，不会收到其他事物的修改的影响，会受到删除和插入影响（注意无修改）。解决了脏读和不可重复读的问题，但还会遇见幻读的问题。因为事务开的时候会创建一个快照（快照只保证了事物读的一致性）。

###  SERIALIZABLE（可串行化）

​	最严格的隔离级别，事务串行执行，保证了数据的一致性、避免了脏读、不可重复读和幻读的问题。

## 传播级别

### REQUIRED （默认）

如果当前存在事物那就加入事物，如果没有事物就创建新事物

### SUPPORTS

​	如果当前存在事物那么久加入当前事物，如果没有事物则只以非事物方式执行。

## MANDATORY

​	要求当前存在事物，否则抛出异常

### REQUIRES_NEW

​	创建一个新事物，并刮起当前事物（外部方法会等内部方法执行完毕后执行）

### NOT_SUPPORTED	

​	以非事物的方式执行，如果存在事物就挂起它，当之前的事务执行完毕就会执行挂起的事务。

### NEVER

​	以非事物方法执行，如果当前存在事物就抛出异常

## @Transaction

他的作用范围在方法（只可以是public）类上和接口上。

他的参数可以设置传播行为默认为required、事务隔离级别是底层数据库的默认事务级别，

### 为什么要求方法必须是public

​	因为@Transaction的基于AOP实现， AOP的工作原理决定了公共方法才能被代理。

### 为什么不推荐在接口上使用

​	因为接口是定义行为和契约的地方，它描述了一个类应该有的方法和行为，它不关心具体的实现细节。但是该注解是和具体实现相关的。

## 因为SpringAOP工作原理导致@Transaction失效 

​	当一个方法被标记了@Transactional注解的时候，Spring事务管理器只会在被其他类方法调用的时候生效，而不会在一个类中方法调用生效。

​	这是因为Spring AOP工作原理决定的。因为Spring AOP使用动态代理来实现事务的管理，他会在运行的时候为带有@Transaction注解的方法生成代理对象，并在方法调用的前后应用事物逻辑。如果该方法被其他类调用我们的代理对象就会拦截方法调用并处理事物。但是在一个类中的其他方法内部调用的时候我们代理对象就无法拦截到这个内部调用，因此事物也就失效了。	
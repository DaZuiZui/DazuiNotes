# Mybatis的工作原理（工作组件）

他的核心原理就是通过解析配置文件生成配置对象，然后利用动态代理创建Mapper接口的代理对象，然后使用Executor执行SQL语句并处理结果集，然后最终将查询结果映射到Java对象中。

## 配置文件解析

Mybatis通过XMl配置文件或注解配置映射关系，这些配置文件定义了数据库的连接信息、SQL映射语句、结果映射等内容。

解析这些配置文件时Mybatis启动的时候首先做的操作，解析完成后会生成相应的配置对象。

## SqlSessionFactory和SqlSession

### SqlSessionFactory

SqlsessionFactory接口用于创建SqlSession实列。

### SqlSession

SqlSession是Mybatsi与数据库交互的主要接口，提供了执行SQL命令、获取映射器、管理实务等功能。

## 动态代理

Mybatis使用java的动态代理（Dynamic proxy）来实现接口（Mapper）方法与SQL语句绑定。

Mapper接口中每个方法对应一条SQL语句，代理对象负责在调用接口方法时执行相应的SQL语句。

## Excutor执行器

Mybatis提供了多种Excutor执行期（SimpleExcutor、ResuseExecutor、RatchExecutor）用来执行sql语句。

excutor负责管理SQL语句的执行，查询结果的缓存、事物的处理等。

### SimpleExecutor

每次执行SQL语句的时候，他都会打开一个新的PreparedStatement对象（用于执行预编译的SQL语句）。

**特点**

​	每次执行SQL语句都会创建一个新的PreparedStatement对象。

​	执行完sql语句后就会关闭PreparedStatement对象

​	不会出重用任何已经创建的PreparedStatement对象。

**适合的场景**

​	适用于简单的，不需要频繁执行相同的SQL语句的场景。

​	适合事物范围内只执行少量的SQL语句

### ReuseExecutor

ReuseExecutor在执行SQl语句的时候，会重用已创建PreparedStatement对象。

**特点：**

​	如果同一SQL语句被多次执行，他就会用上次创建的PreparedStatement对象，而不是每次都创建新的对象。

​	在整个SqlSession生命周期内，PreparedStatement对象只在需要的时候关闭，比如我不用了。

**适合场景：**

​	适用于同一个SqlSession中多次执行相同SQL语句的场景。

​	能够减少PreparedStatement、对象的创建和关闭开销，提高性能。

### Batchexecutor

BatchExecutor采用批量执行的策略来执行SQL。

**特点：**

多天SQl语句缓存到一个批量处理中，等待批处理达到一定条件（如数量、大小）的时候，一次性执行所有缓存的SQL语句。只有调用excutebatch方法时候才会执行。、

**适合场景：**

比较适合批量插入、批量更新删除场景。能明显的减少数据库的网络交互次数，提高批量操作的性能。

## MappedStatement和BoundSQL

MappedStatement是Mybatis内部对象，表示一条sql的完整信息，包括sql语句、输入参数，输出结果。

BoundSQL事SQL语句与参数绑定后的最终执行SQL	

## 类型处理器

Mybatis提供了丰富的类型处理器（TypeHandler），用于将Java类型与JDBC类型相互转换。

默认情况下Mybatis提供了一些常用类型的处理器，也可以自定义类型处理器。

## 结果映射集

Mybatis支持将查询结果映射到Java对象中，可以通过XML配置文件或者注解定义映射关系。

结果集映射器会将查询结果自动转换为相应的Java对象，简化了数据处理的过程。

## 缓存机制

### SqlSession 一级缓存

作用域在一个SqlSession中有效，只要在一个SqlSession中多次查询相同的查询，并且参数相同就会命中缓存。

当Sqlsession结束了，缓存也就会被清理

## mapper 二级缓存

耳机缓存的作用范围事Mapper映射器，同一个Mapper中查询可以共享数据，跨SqlSession共享缓存结果,

当缓存容量达到预设的最大值时或者到达了刷新时间二级缓存就会被清理。	

~~~xml
<cache flushInterval="60000"/> <!-- 每60秒刷新一次 -->
<cache size="512"/> <!-- 最大缓存对象数 -->

~~~


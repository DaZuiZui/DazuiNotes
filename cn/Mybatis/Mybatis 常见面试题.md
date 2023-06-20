# Mybatis 常见面试题

### xml 映射文件中，除了常见的 select、insert、update、delete 标签之外，还有哪些标签？

​	<resultMap> 用来定义结果映射，将查询结果映射到对象或者对象属性上

​    <sql>  用来定义可重用的SQL片段，可以在其他语句中引用	

  <include> 用于包含其他SQL片段或者饮用外部SQL文件

 <if> 用于条件判断，是动态SQL的一部分

 <choose> <when> <otherwise> 用于多重条件判断，类似于Java中的switch语句。

<foreach>用于循环遍历集合或者数组，并插入SQL语句中

<bind> 用于给参数赋值，并将赋值结果作为遍历在后续SQL语句中使用。

~~~xml
<select id="getDiscountedProducts" resultType="Product">
  <bind name="discountedPrice" value="originalPrice * 0.8"/>
  SELECT * FROM products WHERE price > #{discountedPrice}
</select>
~~~

<result> 用来定义结果映射的规则，将数据库查询的结果映射到Java的对象的属性

<association> 用于定义对象关联关系，将多个结果集按照关联关系组合成一个结果对象

<collection>  将多个结果集映射到一个集合属性中

## 动态sql有什么





## #{}和${}有什么区别

​	${} 是静态文本替换 

​	#{}Mybatis会将sql中的#{}替换为？，在sql执行前会使用PreparedStatement的参数设置方法，给sql的？替换为占用符设置的，，#{}的取值方式是通过反射从对象获取到参数的

### Dao 接口的工作原理是什么？Dao 接口里的方法，参数不同时，方法能重载吗？

​		Dao接口的工作原理就是定义一些接口来描述对数据源的访问的操作，然后通过Dao的实现类去实现这些接口的方法，Dao通常都是对数据源的增删改查以及其他的特定业务需求的方法。

​		如果参数不同是可以的，因为因为它可以根据不同的参数顺序和参数执行不同的业务需求。

​		Dao主要就是对数据源的访问操作进行了封装，它提供了一种将业务逻辑与数据访问逻辑分离的方式，这样我们的代码更方便维护和测试。

### 简述 MyBatis 的插件运行原理，以及如何编写一个插件

​	Mybatis插件的运行原理就是基于Java的动态代理和拦截器实现的。

​	当执行层安讯更新等数据库操作时，Mybatis会通过动态代理来生成相应的代理对象。

​	插件会拦截这些代理对象的方法调用，并在调用前后插入自定义的逻辑。

编写的步骤就是主要针对Parameterhander、ResultSetHanlder、StatementHandler、Executor这四个接口。	

​	

​	实现Interceptor接口的插件类，该接口定义了插件的核心方法intercept，和插件的包装目标对象方法plugin方法对对象进行包装返回代理对象，然后再Mybatis配置文件中注册插件，指定插件的拦截器路径和配置参数。

### MyBatis 执行批量插入，能返回数据库主键列表吗？

可以，因为Mybatis的底层是JDBC。

### MyBatis 动态 sql 是做什么的？能简述一下动态 sql 的执行原理不？

​	动态SQL就是根据不同的条件自动生成不同的SQL语句。

他的执行原理就是

​	**解析构造阶段：**在Mybatis解析配置文件的时候会解析动态SQL，并根据不同的条件构建响应的SQL语句片段。

​	**SQL语句合并阶段：**在构造完整的SQL语句时，Mybatis会将动态SQL中的各个片段进行合并。

​	**参数绑定阶段，**Mybatis会根据动态sql 的参数占位符选择合适的方法获取值（如#{}就是反射获取，）进行参数绑定。

​	**SQL语句执行阶段：**Mybatis会将最红的SQL语句发送给数据库执行， 并获取结果
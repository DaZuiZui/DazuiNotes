# Spring中的切点（Pointcut）是什么？它的作用是什么，如何定义一个切点？

 	切点是Spring aop中的一个定义，用于确定哪些方法或位置需要应用通知，它的作用就是指定切面的工作范围，告诉AOP什么时候触发通知。实现切点我们可以使用

​	**AspectJ：**

​			使用该方法的优点是，AspectJ提供了很高的灵活性，可以通过表达式找到非常精确的切点、类、参数等。

表达式参数：

~~~java
execution(modifiers-pattern? return-type-pattern declaring-type-pattern? method-name-pattern(param-pattern) throws-pattern?)
~~~

- `modifiers-pattern`：方法的访问修饰符，如 `public`、`protected`、`private`。
- `return-type-pattern`：方法的返回类型。
- `declaring-type-pattern`：方法所属的类或接口。
- `method-name-pattern`：方法的名称。
- `param-pattern`：方法的参数列表。
- `throws-pattern`：方法可能抛出的异常。

​	**Pointcut:**

​			使用该方法的优点是，它简单明了，基于注解风格，代码容易维护和理解。

​				            但是缺点是,  切点和切面是强耦合，不够灵活，难以在不同切面共享切点，导致代码重复

​	**XML**	

​			使用该方法的优点，可以让我们多个节点共享切点，让我们切点从切面分割，让我们更容易维护

​					缺点是：需要写大量的XML代码
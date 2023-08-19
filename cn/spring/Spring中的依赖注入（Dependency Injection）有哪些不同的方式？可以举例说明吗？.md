# **Spring中的依赖注入（Dependency Injection）有哪些不同的方式？可以举例说明吗？**

## 构造方法注入

 使用构造函数进行DI注入

~~~java
public class MyClass {
    private MyDependency myDependency;

    public MyClass(MyDependency myDependency) {
        this.myDependency = myDependency;
    }
}
~~~

## setter方法注入

使用set方法进行注入

~~~java
public class MyClass {
    private MyDependency myDependency;

    public void setMyDependency(MyDependency myDependency) {
        this.myDependency = myDependency;
    }
}

~~~

## 接口注入

接口注入是通过目标类实现注入接口并在接口方法中进行依赖注入的方式，这种方法不太常见。

~~~java
public interface MyInterface {
    void injectDependency(MyDependency myDependency);
}

public class MyBean implements MyInterface {
    private MyDependency myDependency;

    @Override
    public void injectDependency(MyDependency myDependency) {
        this.myDependency = myDependency;
    }

    // 其他方法...
}

~~~

## 注解注入

比如使用autowired和component注解

~~~java
@Component
public class MyBean {
    private MyDependency myDependency;

    @Autowired
    public MyBean(MyDependency myDependency) {
        this.myDependency = myDependency;
    }

    // 其他方法...
}
~~~

## XML注解

定义一个BEAN标签，id是我们bean的id，class是我们目标的路径，如果我们使用setter方法进行注入就property标签，name是我们的字段名字，value是我们的值，如果已经配置依赖注入就把value变成ref，如果我们使用如果使用构造方法就使用constructor-arg 

~~~java
<bean id="myBean" class="com.example.MyBean">
    <property name="name" value="John Doe" />
    <property name="age" value="30" />
</bean>
<bean id="myBean" class="com.example.MyBean">
    <property name="name" value="John Doe" />
    <property name="age" value="30" />
</bean>
<bean id="myBean" class="com.example.MyBean">
    <constructor-arg index="0" value="John Doe" />
    <constructor-arg index="1" value="30" />
</bean>
<bean id="myBean" class="com.example.MyBean">
    <property name="address" ref="myAddressBean" />
</bean>

<bean id="myAddressBean" class="com.example.Address" />

~~~


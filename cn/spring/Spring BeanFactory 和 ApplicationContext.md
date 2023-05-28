# Spring BeanFactory 和 ApplicationContext

## BeanFactory

​	Beanfactory是Spring IOC的基本接口，他定义了容器最基本的功能，提供了对Bean的注册、获取和管理能力。

​	BeanFactory使用延迟初始化策略，也就是说，当你从容器中获取一个Bean时，才会真正的创建和初始化Bean。

​	BeanFactory对于资源的消耗比较小，因为它只在需要的时候才会创建Bean对象，适合资源受限的环境。

​	DefaultListableBeanfactory是BeanFactory接口的一个常见实现类，提供了对Bean的定义、解析和注册的功能。

### BeanFactory对AOP的支持

~~~java
// 创建BeanFactory实例
BeanFactory beanFactory = new DefaultListableBeanFactory();

// 创建切面类
@Aspect
public class MyAspect {
    @Before("execution(* com.example.MyService.doSomething())")
    public void beforeAdvice() {
        // 执行前置通知逻辑
        System.out.println("Before advice executed");
    }
}

// 创建目标类
public class MyService {
    public void doSomething() {
        // 执行业务逻辑
        System.out.println("Doing something");
    }
}

// 注册切面和目标类
BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
registry.registerBeanDefinition("myAspect", new RootBeanDefinition(MyAspect.class));
registry.registerBeanDefinition("myService", new RootBeanDefinition(MyService.class));

// 创建AOP代理对象
AopProxyFactory proxyFactory = new DefaultAopProxyFactory();
Advisor[] advisors = new Advisor[]{new DefaultPointcutAdvisor(new AspectJExpressionPointcut(), new MyAspect())};
Object target = beanFactory.getBean("myService");
Object proxy = proxyFactory.createAopProxy(target, advisors).getProxy();

// 将AOP代理对象注册到BeanFactory中
beanFactory.registerSingleton("myServiceProxy", proxy);

// 获取AOP代理对象
MyService myService = beanFactory.getBean("myServiceProxy", MyService.class);

// 调用目标方法
myService.doSomething();

~~~

BeanFactory需要做一些手动配置，如AOP代理对象进行绑定。 很麻烦

## DefaultListableBeanFactory

​	上文我们所说到DefualtListableBeanFactory提供了Bean的定义、解析和注册的功能。

### Bean的定义

​	**Bean的定义描述了如何创建和配置一个特定的Bean。它通常包括Bean的类名（Bean的class）、依赖关系、属性值和初始化方法等信息。**

~~~xml
<bean id="myBean" class="com.example.MyBeanClass">
    <!-- Bean的属性和配置 -->
</bean>

~~~

**在这个例子中，"com.example.MyBeanClass" 是要实例化的Bean的类名。**

#### 定义Bean的类

 就是创建一个Java类，表示我们要创建的Bean，这个类可以包括属性、方法和构造函数以满足特定的业务需求。

~~~java
public class Builder {
    private Long id;
    private String name;
    private Integer maxFloor; //最大楼层
    private String sex;      //所属性别

    @Override
    public String toString() {
        return "Builder{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", maxFloor=" + maxFloor +
                ", sex='" + sex + '\'' +
                '}';
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getMaxFloor() {
        return maxFloor;
    }

    public void setMaxFloor(Integer maxFloor) {
        this.maxFloor = maxFloor;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public Builder() {
    }

    public Builder(Long id, String name, Integer maxFloor, String sex) {
        this.id = id;
        this.name = name;
        this.maxFloor = maxFloor;
        this.sex = sex;
    }
}

~~~

##### 注册声明Bean

​	声明bean主要做的就是指定Bean的类名、属性值和依赖关系。

###### xml配置文件声明bean

~~~xml
<!-- 声明一个名为 "userService" 的Bean，类为 "com.example.UserService" -->
<bean id="userService" class="com.example.UserService">
   <!-- 设置属性值 -->
   <property name="name" value="John Doe" />
   <property name="age" value="30" />
   
   <!-- 设置依赖关系 -->
   <property name="userDao" ref="userDao" />
</bean>

<!-- 声明一个名为 "userDao" 的Bean，类为 "com.example.UserDao" -->
<bean id="userDao" class="com.example.UserDao" />

~~~

###### 注解声明Bean

~~~java
@Component
public class UserService {
    @Value("John Doe")
    private String name;
    
    @Value("30")
    private int age;
    
    @Autowired
    private UserDao userDao;
    
    // Bean的其他方法和逻辑...
}
~~~

##### 配置Bean的作用域

###### 注解定义作用域

~~~java
@Component
@Scope("prototype")
public class MyPrototypeBean {
    // Bean的实现代码...
}

~~~

###### xml定义的作用域

~~~java
<bean id="mySingletonBean" class="com.example.MySingletonBean" scope="singleton" />

<bean id="myPrototypeBean" class="com.example.MyPrototypeBean" scope="prototype" />
~~~

##### 配置Bean属性

######  注解

~~~java
@Component
public class MyBean {
    @Value("John Doe")
    private String name;
    
    @Value("30")
    private int age;
    
    // Getter and Setter methods...
}

~~~

###### xml

~~~xml
<bean id="myBean" class="com.example.MyBean">
    <property name="name" value="John Doe" />
    <property name="age" value="30" />
</bean>
~~~

### 解析

​	Bean的解析是指Bean 的定义转移为容器内部结构的过程，方便容器能够理解管理这些Bean，在解析过程中，容器会读取和解析XML配置文章、扫描并解析Java注解或处理Java代码，以获取Bean的定义信息。解析过程中容器会创建对应的数据结构（如BeanDefinition对象）来表示每个Bean的定义，并将其粗处容器的内部数据结构中。

​	定义初始化方法也就是在解析阶段做的，定义初始化方法就是销毁前，初始化方法。

​	当Spring启动的时候，他就会解析配置文件或者注解中，创建和注册的Bean，这意味着spring会是实列化该Bean，并根据配置文件中的设置来注入依赖关系和属性值。

### 注册

​	Bean的注册是将解析到的Bean定义注册到容器中的过程，这样容器就能够管理这些Bean，并在需要的时候进行创建和管理。

​	在注册过程中，容器将解析到的Bean定义与其一的标识符（通常Bean的name和id）相关联，并将其存储在容器的Bean注册表中。

​	注册的方法取决于使用的配置方式，在XML配置文件中你可以使用<bean>元素的id或者name属性来指定Bean的标识符。在使用Java注解或Java代码时，通常使用注解元数据或方法名作为Bean的标识符。如果id和name同时出现那么id优先级最高，因为name只是一个别名。

### 获取Bean实列

​	一旦Bean被注入到容器，就可以通过容器获取到该Bean。



## BeanDefinition

​	上面我们在解析的时候说到，在解析的时候会创建Definition对象，它包含了描述bean的元数据信息，例如Bean的类名、作用域、构造函数参数、属性值、初始化方法、销毁方法等。他主要做的就是用于描述和管理Bean的配置和创建过程。

​	每个注册到Spring容器的bean都对应一个的BeanDefinition对象，当Spring容器扫描注解或者去读和解析xml配置文件的时候他就会将这些配置信息创建到对应的BeanDefinition对象。

​	通过BeanDefinition，Spring容器可以了解Bean的特征和配置细节，以便在需要的时候创建的、管理和使用Bean。BeanDefinition的信息可以让我们的Spring容器能狗实现依赖注入、AOP（需要BeanDefinition和Aop配置文件结合，spring会根据AOP配置判断哪些Bean需要创建AOP代理对象，并将AOP的横切逻辑织入到这些Bean的方法调用中。）、生命周期管理等功能。

​	BeanDefinition的实现类有GenericBeanDefinition和RootBeanDefinition

### BeanDifnition与AOP的结合

​	BeanDefinition与AOP的结合是通过AOp相关的配置元素或者注解实现的。BeanDefinition中PropertyValues属性可以用来设置Bean的属性值，包括AOP相关的配置信息。可以使用propertyValues来设置切换（Aspect）、切入点（Poinicut）和通知（Advice）对象等

### GenericBeanDefinition

默认情况下BeanDefinition的实现类都是GenericBeanDefinition

#### 属性设置

​	它可以设置Bean的各种属性、包括类型、作用域、构造函数参数、属性值等。

#### 依赖关系管理

​	GenericBeanDefinition可以声明Bean之间的依赖关系，以实现依赖注入。

#### 初始化和销毁方法

​	它可以指定Bean在创建后需要执行的初始化方法和在销毁前执行的方法、

### RootBeanDifinition

​	RootBeanDIfinition是BeanDifinition的另外一个实现类，他可以贡献作用域（Scope）和初始化方法（init-method）等配置属性。

~~~java
@Configuration
public class AppConfig {
    @Bean
    public RootBeanDefinition myBeanDefinition() {
        RootBeanDefinition beanDefinition = new RootBeanDefinition(MyBean.class);
        // 设置其他属性或依赖关系
        beanDefinition.setScope("singleton");
        beanDefinition.setInitMethodName("init");
        beanDefinition.setDestroyMethodName("destroy");
        return beanDefinition;
    }

    @Bean
    public MyBean myBean() {
        return new MyBean();
    }
}

~~~

## ApplicationContext

​	ApplicationContext是BeanFactory的子接口主要提供了更多的功能和扩展性和便捷性。

​	ApplicationContext负责管理和维护Bean的声明周期，主要提供了以下的功能

**1.Bean的创建和管理：**ApplicationContext负责实例化、配置和管理Bean，他会根据xml文件或者注解解析Bean的定义根据需求创建Bean。

**2.依赖注入：ApplicationContext可以自动解决Bean之间的依赖关系，它会根据Bean的定义信息，自动将依赖的bean注入到目标Bean中，实现解耦和组件之间的协作**

**3.AOP支持：**可以配置和管理切面和切入点和通知。

**4.事件的发布和监听：就是APplicationContext发布一条信息让已注册的监听器让他们执行相应的处理操作**

​	创建一个自定义事件类：这个自定义类要继承ApplicationEvent

~~~java
public class MyCustomEvent extends ApplicationEvent {
    // 自定义事件类的构造方法
    public MyCustomEvent(Object source) {
        super(source);
        // 可以在这里设置事件相关的属性
    }
    // 自定义事件类的其他方法和属性
}
~~~

然后通过ApplicationContext.publishEvent();方法进行发布事件

事务的监听实现ApplicationListener<T>进行监听

~~~java
@Component
public class MyCustomEventListener implements ApplicationListener<MyCustomEvent> {
    @Override
    public void onApplicationEvent(MyCustomEvent event) {
        // 在这里编写事件处理逻辑
        // 可以访问事件相关的数据和信息
    }
}

~~~

**5.国际化支持：**ApplicationContext提供国际化的支持，可以根据不同的语言和地区，加载相应的资源文件

**6.生命周期管理：**ApplicationContext管理Bean的声明周期，包括Bean的初始化、销毁、和回收。他会在合适的时候调用Bean的生命周期回调方法，确保Bean在创建和销毁的时候执行必要的操作。

### ApplicationContext对Aop的支持

​	虽然BeanFactory也支持AOP，但是在实际使用中ApplicationContext更加便捷，ApplicationContext内部继承了AOP相关的功能，包括切面、切点、通知等，可以直接使用注解或或者配置来实现Aop相比之下BeanFacotry需要手动配置和管理AOP的相关组件，比较麻烦。

​	ApplicationContext和BeanFactory比起来它更加的方便，不需要手动AOP代理对象的绑定，它提供了自动扫描和解析AOP的能力**@EnableAspectAutoProxy**

~~~java
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;
import org.springframework.context.annotation.aspectj.EnableSpringConfigured;
import org.springframework.stereotype.Component;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Configuration
@EnableAspectJAutoProxy
@EnableSpringConfigured
public class AppConfig {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        
        MyBean myBean = context.getBean(MyBean.class);
        myBean.doSomething();
        
        context.close();
    }
}

@Component
class MyBean {
    public void doSomething() {
        System.out.println("Doing something...");
    }
}

@Aspect
@Component
class MyAspect {
    @Before("execution(* com.example.MyBean.doSomething())")
    public void beforeAdvice() {
        System.out.println("Before advice executed");
    }
}

~~~



#### @EnableAspectAutoProxy

​	@EnableAspectAutoProxy通过在配置类上添加该注解，我们可以启动Spring框架对AOP注解和自动扫描和解析。就可以通过注解自动进行AOP对象的绑定，不需要我们手动配置

​	@EnableAspectAutoProxy注解告诉Spring容器要使用AspectJ自动代理功能来实现AOP，他会扫描应用上下中的Bean，并检查他们是否有被@Aspect注解标记切面类，如果发现了切面类，它就会为这些切面创建代理对象，并将他们应用到合适的目标bean上，如果遇见AOP相关的注解@before、@After那么Spring就会扫描这些注解，并且为他们做相应的切面逻辑。

​	@EnableAspectAutoProxy提供了自动扫描和解析AOP注解的能力，使得我们更加方便的配置和使用AOP功能。

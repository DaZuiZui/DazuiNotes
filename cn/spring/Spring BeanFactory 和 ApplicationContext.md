# Spring BeanFactory 和 ApplicationContext

## BeanFactory

    Beanfactory是Spring IOC的基本接口，他定义了容器最基本的功能，提供了对Bean的注册、获取和管理能力。
    
    BeanFactory使用延迟初始化策略，也就是说，当你从容器中获取一个Bean时，才会真正的创建和初始化Bean。
    
    BeanFactory对于资源的消耗比较小，因为它只在需要的时候才会创建Bean对象，适合资源受限的环境。
    
    DefaultListableBeanfactory是BeanFactory接口的一个常见实现类，提供了对Bean的定义、解析和注册的功能。

### BeanFactory对AOP的支持

```java
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

```

BeanFactory需要做一些手动配置，如AOP代理对象进行绑定。 很麻烦

## DefaultListableBeanFactory

    上文我们所说到DefualtListableBeanFactory提供了Bean的定义、解析和注册的功能。

### Bean的定义

    **Bean的定义描述了如何创建和配置一个特定的Bean。它通常包括Bean的类名（Bean的class）、依赖关系、属性值和初始化方法等信息。**

```xml
<bean id="myBean" class="com.example.MyBeanClass">
    <!-- Bean的属性和配置 -->
</bean>

```

**在这个例子中，"com.example.MyBeanClass" 是要实例化的Bean的类名。**

#### 定义Bean的类

 就是创建一个Java类，表示我们要创建的Bean，这个类可以包括属性、方法和构造函数以满足特定的业务需求。

```java
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

```

##### 注册声明Bean

    声明bean主要做的就是指定Bean的类名、属性值和依赖关系。

###### xml配置文件声明bean

```xml
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

```

###### 注解声明Bean

```java
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
```

##### 配置Bean的作用域

###### 注解定义作用域

```java
@Component
@Scope("prototype")
public class MyPrototypeBean {
    // Bean的实现代码...
}

```

###### xml定义的作用域

```java
<bean id="mySingletonBean" class="com.example.MySingletonBean" scope="singleton" />

<bean id="myPrototypeBean" class="com.example.MyPrototypeBean" scope="prototype" />
```

##### 配置Bean属性

###### 注解

```java
@Component
public class MyBean {
    @Value("John Doe")
    private String name;
  
    @Value("30")
    private int age;
  
    // Getter and Setter methods...
}

```

###### xml

```xml
<bean id="myBean" class="com.example.MyBean">
    <property name="name" value="John Doe" />
    <property name="age" value="30" />
</bean>
```

### 解析

    Bean的解析是指Bean 的定义转移为容器内部结构的过程，方便容器能够理解管理这些Bean，在解析过程中，容器会读取和解析XML配置文章、扫描并解析Java注解或处理Java代码，以获取Bean的定义信息。解析过程中容器会创建对应的数据结构（如BeanDefinition对象）来表示每个Bean的定义，并将其粗处容器的内部数据结构中。
    
    定义初始化方法也就是在解析阶段做的，定义初始化方法就是销毁前，初始化方法。
    
    当Spring启动的时候，他就会解析配置文件或者注解中，创建和注册的Bean，这意味着spring会是实列化该Bean，并根据配置文件中的设置来注入依赖关系和属性值。

### 注册

    Bean的注册是将解析到的Bean定义注册到容器中的过程，这样容器就能够管理这些Bean，并在需要的时候进行创建和管理。
    
    在注册过程中，容器将解析到的Bean定义与其一的标识符（通常Bean的name和id）相关联，并将其存储在容器的Bean注册表中。
    
    注册的方法取决于使用的配置方式，在XML配置文件中你可以使用`<bean>`元素的id或者name属性来指定Bean的标识符。在使用Java注解或Java代码时，通常使用注解元数据或方法名作为Bean的标识符。如果id和name同时出现那么id优先级最高，因为name只是一个别名。

### 获取Bean实列

    一旦Bean被注入到容器，就可以通过容器获取到该Bean。

## BeanDefinition

    上面我们在解析的时候说到，在解析的时候会创建Definition对象，它包含了描述bean的元数据信息，例如Bean的类名、作用域、构造函数参数、属性值、初始化方法、销毁方法等。他主要做的就是用于描述和管理Bean的配置和创建过程。
    
    每个注册到Spring容器的bean都对应一个的BeanDefinition对象，当Spring容器扫描注解或者去读和解析xml配置文件的时候他就会将这些配置信息创建到对应的BeanDefinition对象。
    
    通过BeanDefinition，Spring容器可以了解Bean的特征和配置细节，以便在需要的时候创建的、管理和使用Bean。BeanDefinition的信息可以让我们的Spring容器能狗实现依赖注入、AOP（需要BeanDefinition和Aop配置文件结合，spring会根据AOP配置判断哪些Bean需要创建AOP代理对象，并将AOP的横切逻辑织入到这些Bean的方法调用中。）、生命周期管理等功能。
    
    BeanDefinition的实现类有GenericBeanDefinition和RootBeanDefinition

### BeanDifnition与AOP的结合

    BeanDefinition与AOP的结合是通过AOp相关的配置元素或者注解实现的。BeanDefinition中PropertyValues属性可以用来设置Bean的属性值，包括AOP相关的配置信息。可以使用propertyValues来设置切换（Aspect）、切入点（Poinicut）和通知（Advice）对象等

### GenericBeanDefinition

默认情况下BeanDefinition的实现类都是GenericBeanDefinition

#### 属性设置

    它可以设置Bean的各种属性、包括类型、作用域、构造函数参数、属性值等。

#### 依赖关系管理

    GenericBeanDefinition可以声明Bean之间的依赖关系，以实现依赖注入。

#### 初始化和销毁方法

    它可以指定Bean在创建后需要执行的初始化方法和在销毁前执行的方法、

### RootBeanDifinition

    RootBeanDIfinition是BeanDifinition的另外一个实现类，他可以贡献作用域（Scope）和初始化方法（init-method）等配置属性。

```java
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

```

## ApplicationContext

​	ApplicationContext主要对BeanFactory做了一些扩展如

​	**1.Aop支持**，Appplication可以更加方便的配置和使用AOP切面，可以进行AOP动态代理，也支持了AOP配置支持，也集成了AOP框架，也可以使用BeanPostProcessor让我们的在Bean在初始化过程中自定义AOP代理的创建和设置，内置了**AOP**，如ApplicationContext内置了AOP框架，可以方便与其他SPring功能集成，如@Transaction。

​	**2.国际化支持**，可以根据环境语言选择不同的

​	**3.事务的发布与监听，**它可以让Veab发布事件让其他Bean去接受这个事件。

​	**4.资源加载:**他可以让我们更加方便的加载功能，可以加载各个类型的资源、如文件和类路径、URL等，可以让我们方便轻松的访问和管理外部资源。BeanFacotry只能通过XML进行加载，ApplicationContext支持更多类型的资源还支持了自动刷新也就是当文件进行更新我们的Bean也会自动更新。

​	**5.生命周期的管理**，在实列化和初始化和销毁阶段都做了响应的回调方法，以便必要的时候进行操作。

​	**6.注解的配置**，支持了注解，如Autowried

​	**7.MVC的集成，**ApplicationContext提供了MVC的集成

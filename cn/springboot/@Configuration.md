## @Configuration

@Configuration就是把这个类作为一个bean，然后把我这个类下的所有bean都注入到ioc容器中



因为Spring把@Configuration类也注册为Bean就是为了让Spring能代理它，拦截他的@Bean方法，确保依赖注入的工作正常。

| 为什么 `@Configuration` 本身也要作为 Bean？              | 原因                |
| -------------------------------------------------------- | ------------------- |
| 1️⃣ 能被 Spring 管理                                       | 支持依赖注入        |
| 2️⃣ 支持 `@Bean` 方法之间相互调用时仍保持单例              | 通过 CGLIB 代理拦截 |
| 3️⃣ 支持高级场景（如 `@Import`, `@DependsOn`, 运行时注入） | 灵活配置框架行为    |

## 深入理解

因为这个配置类会被代理的（CGLIB代理），当我们Spring在处理这个类的时候，会创建他的子类代理对象（而不是直接用你的类）。

因为在这个类内部调用另外一个bean的方法的时候，他能拦截这个调用，从Spring容器里返回已经创建好的bean而不是重新new一个。

~~~java
@Configuration
public class MyConfig {

    @Bean
    public A a() {
        return new A(b()); // 你以为你在拿 B 其实没有
    }

    @Bean
    public B b() {
        return new B();
    }
}

~~~


# @Lazy解决循环依赖

## what is lazy

lazy就是用来延迟初始化Bean的，通常Spring会在启动的时候立即初始化所有的bean，如果我们使用啦Lazy，那么这个bean就会在真正使用的时候才会被初始化。

所以他的使用场景有减少**启动时的资源消耗，解决循环依赖的问题**。

## 如何解决的循环依赖

review this code

~~~java
@Component
public class BeanA {
    private final BeanB beanB;

    @Autowired
    public BeanA(BeanB beanB) {
        this.beanB = beanB;
    }
}

@Component
public class BeanB {
    private final BeanA beanA;

    @Autowired
    public BeanB(BeanA beanA) {
        this.beanA = beanA;
    }
}
~~~

当我们A依赖于B，B依赖A这样就形成了一个循环依赖，这种情况下，Spring无法决定创建哪个bean，最终会导致一个异常。

这时候我们就可以使用@lazy，这样Spring不会立即尝试解决这个依赖，而是在第一次访问该Bean的时候才会进行注入。

他的工作原理就是当Spring容器的时候会先创建BeanA，但是BeanA依赖的BeanB是延迟加载的，所以BeanB不会立即被初始化。

当BeanA实际调用BeanB的时候，Spring会在那时候创建并注入beanB，同时将BeanA作为参数给我们的BeanB。这样就打破了循环依赖，因为在BeanA完全创建之前，beanB并没有被真正的实列化。
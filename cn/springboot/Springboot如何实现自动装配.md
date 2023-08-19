# Springboot如何实现自动装配

​	自动装配就是把第三方的Bean装载到Spring IOC中，不需要开发人员再去手动写那些Bean的装配配置。

​	**Springboot真正实现自动自动注解的是因为SpringBootApplication里面的EnableAutoConfiguration。**

​		引入Starter启动组件的时候，这个组件必须包含@Configuration配置类，在这个配置类里面通过@Bean注解声明要装配到IOC容器的BEAN对象。

​		这个配置类就是第三方的jar包，然后SpringBoot中约定优于配置思想，会把这个配置类放到spring.factories文件中，这样SPringboot就知道第三方jar包利民啊的配置类的位置。这个步骤主要是Spring里面的SPringFactoriesLoader完成。

​		SpringBoot拿到了第三方jar包，然后在通过SPring提供的ImportSelector接口，实现对这些配置类的动态加载。

​	在我看来Springboot约定大于配置的思想主要让程序员专注于业务层上的开发。
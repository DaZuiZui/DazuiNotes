# Spring核心容器

​	Spring的核心容器主要有BeanFactory和ApplicationContentext。

​	**BeanFactory：**是Spring的核心容器之一，它负责提供应用程序中的对象，也被称为bean，BeanFacotry主要实现了控制反转和基本的依赖注入，它的职责是实力化、配置和管理Beans的生命周期。BeanFactory接口常见的实现类是DefoutListTableBeanfactory。

​	**ApplicationContext：**是BeanFactory的子接口，他是Spring应用程序最高级的容器，它在BeanFactory的基础上添加了很多企业级的特性，包括国际化支持、事件发布、资源加载、AOP集成等，它是我们在实际开发最常用的容器。Application还提供了对环境信息的访问和管理，以便根据不同的配置文件加载不同的Beans。

​	它的实现类主要有**ClassPathXmlApplicationContext**根据XMl配置定义和配置容器和Bean，适用于传统的Spring项目

​									**AnnotationConfigApplicationContext**根据注解来配置容器和bean，适用于现代的Spring项目

​								**FileSystemXmlApplicationContext**从系统路径加载xml文件配置容器和Bean、适用于从系统文件加载xml



BeanFactory的关注点主要是IOC和基本的依赖注入。ApplicationContext主要在此基础提供了更多的企业功能和增强。
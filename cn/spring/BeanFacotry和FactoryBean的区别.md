# BeanFacotry和FactoryBean的区别

## BeanFactory

​	BeanFactory是一个工厂接口，用于管理和创建对象，它负责Bean的生命周期的管理、依赖注入、对象的创建。其BeanFacotry的特点主要有

​		1.提供了通用对象的获取和管理机制。

​				**对象的创建：**负责创建程序中定义的所有Bean对象，可以通XML文件或者注解进行定义。

​				**对象的获取：**可以通过Bean的昵称或类型进行获取。

​				**对象的生命周期：**包括了对象的创建和初始化和使用和销毁。

​				**依赖注入：**可以让对象之间的存在依赖关系进行注入。

​		2.用于管理Bean的生命周期

​		3.支持多种Bean的配置方式、如XML配置、注解配置等。

## FactoryBean

​	FactoryBean是一个特殊的Bean，其实它本身也是一个Bean，他是一个工厂类的封装，用于创建和配置其他的Bean，他是Spring中的一个接口，需要用户自定义实现，如果我们需要在Bean的创建过程中做一些定制化操作或者根据一些条件来决定创建哪种类型的Bean就可以使用FactoryBean。

## 总结

​	BeanFaotry用于通用的管理，FactoryBean是一个用于自定义Bean的创建逻辑使用。
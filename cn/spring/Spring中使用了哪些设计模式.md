# Spring中使用了哪些设计模式

## 单列模式(Singleton Pattern)

单列模式就是保证我们一个类只有一个实例，提供了一个全局的访问点。

## 工厂模式

用来创建对象

## 代理模式

在Aop大量了使用代理模式来增强我们的代码。

## 模版方法

提供了一些固定的操作比如JDBCTeamplate，RestTemplate	

## 观察者模式机制

Spring事件机制使用了观察者模式，ApplicationEvent和ApplicationListener提供了

## 原型

就是我们的bean的scope为prototype

## 装饰者模式

BeanPostProcessor和AOP都可以用来

## 依赖注入

Spring的核心功能就是通过依赖注入来管理对象之间的依赖关系。

## 策略模式

我们Mvc handlerMapping和HandlerAdapater就是使用策略模式来决定选择合适的处理器。
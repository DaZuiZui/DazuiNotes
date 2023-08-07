# Spring Bean如何保证并发安全

## 采用原型（Prototype）模式

​	这样每次从容器获取该Bean的时候都会创建一个新的实列，避免了多线程共享一个对象实列的问题。

## 避免在Bean中存在可变状态的声明

避免bean中存在可变状态的声明，我们可以把可变状态存储到方法中，或者存在ConcurrentHashMap来管理状态

## 使用同步锁

使用synchronized或者ReentranLock来保证同一时刻是有一个线程可以操作。
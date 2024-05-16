# 为什么Spring Cloud gatway不可以引入Spring web

因为Spring Cloud gateway是机遇Spring Webflux构建，而不是传统的Spring Web MVC

## 在设计方面

Spring Cloud gateway设计的目的是应对高并发 高吞吐的场景， 所以采用了SPring WEbFLux的异步非堵塞模型。

而Spring web mvc使用的是传统的同步堵塞模型

## 性能和吞吐量

在高负载情况下，异步可以更好的利用系统资源，提高网关的性能和吞吐量，而同步可能因为线程堵塞性能下降。

## 响应式编程

gatway是机遇响应式编程模型设计，可以方便的处理异步事件流和复杂的路由逻辑，而spring web是传统的请求-响应模式，不太适合处理异步事件流。
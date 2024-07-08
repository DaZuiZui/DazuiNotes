# CompletableFuture

他是jdk8提供的多线程执行工具类，他最大的特色就是支持任务的编排，在保证并发执行， 又能让相互之间有依赖的任务。

## 任务代码

一个任务链的情况

~~~java
ComplatableFutre.supplyAsync(()-> 10).thenApply(r -> r * 1).thenApply(r -> r*2).thenApply(r -> r * 3)
~~~

最后的结果为60。其实completable真正能串联起来的原因是因为ComplatableFutre有2个属性

~~~java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {
  	volatile Object result; //保存这个Future的返回结果
    volatile Completion stack; //用链表实现的栈，保存了所有依赖这个future后续的操作
}
~~~

也就是说每个thenApply都会创建一个新的completion，然后这个completion也会放到他前面的future栈中。
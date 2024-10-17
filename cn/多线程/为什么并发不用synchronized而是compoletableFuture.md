# 为什么并发不用synchronized而是compoletableFuture

因为compoletable是jdk8引入的新工具，他可以帮我写异步、非堵塞的代码，从而避免了传统的synchronized所带来的性能瓶颈，线程堵塞和复杂的锁管理。

## CompletableFuture

使用synchronized回导致线程堵塞，线程需要等锁释放后才能继续执行

CompletableFuture支持异步执行任务，不会堵塞当前线程，任务一旦完成就会自动回调对应的处理逻辑。

~~~java
CompletableFuture.supplyAsync(() -> {
    // 异步任务，模拟耗时操作
    return "Result from async task";
}).thenAccept(result -> {
    // 任务完成后的回调处理
    System.out.println(result);
});

~~~

## 简化回调逻辑，避免回调地狱

在传统的异步编程中（通过线程池或回调函数），代码容易变得复杂和嵌套（回调地狱），让异步更加清晰

~~~java
CompletableFuture.supplyAsync(() -> {
    return "Task 1";
}).thenApply(result -> {
    return result + " -> Task 2";
}).thenAccept(finalResult -> {
    System.out.println(finalResult);
});
~~~

**输出**：`Task 1 -> Task 2`
通过链式调用，多个任务按顺序执行，代码保持简洁。

## 更高的性能：非堵塞 vs 堵塞

synchronized会导致线程堵塞，多个线程竞争产生的上下文切换，影响性能。

CompletableFuture使用Forkjoinpool可以实现轻量级的线程任务调度，减少上下文切换。

## 并行处理

completablefuture提供了多种任务组合的方式，可以让多个任务并行执行，并在所有任务完成后处理结果。

~~~java
CompletableFuture<String> task1 = CompletableFuture.supplyAsync(() -> "Task 1");
CompletableFuture<String> task2 = CompletableFuture.supplyAsync(() -> "Task 2");

task1.thenCombine(task2, (result1, result2) -> {
    return result1 + " and " + result2;
}).thenAccept(System.out::println);

~~~


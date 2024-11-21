# 为什么不推荐使用ExcutorService来创建线程

## 1.线程管理复杂性

如果直接使用ExcutorSerivce来创建线程，而不管理线程池的大小和生命周期可能会导致我们的线程无限制增长。

## 2.线程池滥用的风险

ExcutorService需要用户去调用shudown和shutdownNow来释放资源，如果没有正确关闭线程池可能会一直存在，导致资源的泄漏。

## 3.隐形的增加程序的复杂性。

如果不明确指定线程池的配置，比如核心线程数和最大线程

所以使用我们线程池的工具类ThreadPoolExecutor或者使用CompletableFuture来处理并发任务。
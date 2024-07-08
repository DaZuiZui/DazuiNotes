# 内存队列是否存在oom问题

## what is 内存队列

内存队列一般是java.util.concurrent下的线程安全的包，比如ArrayblockingQueue。

## 是否存在oom？

答案是存在的，因为内存队列是使用JVM堆，如果里面元素太多了， 内存不够用了就会出现oom报错。
# 深入理解 Java 中的 `volatile` 关键字：可见性与有序性的保障

volatile主要做了两个事情**可见性保证** 和 **有序性**

**可见性保证就是：** 对volatile的写操作会对其他线程可见。

简单来说我们A线程的修改了volatile的值，那么我B线程也可以看见。

**有序性：** 有序性就是他确保了一个对volatile之前所有的写操作，不可以被cpu重排序，哪怕是编译阶段。

简单来说

~~~java
int a = 0;
volatile int v = 0;
a = 1;
v = 1;
~~~

volatile确保了让CPU先执行a=1彻底写进主内存然后在执行v=1

## 为什么一定要保证有序性？

因为在多线程编程有序性是确保操作按照预期执行的重要机制。

~~~java
class Example {
    int a = 0;
    volatile boolean flag = false;

    public void writer() {
        a = 1;           // 1
        flag = true;     // 2
    }

    public void reader() {
        if (flag) {      // 3
            int i = a;   // 4
            // i should be 1
        }
    }
}

~~~

这个线程一个执行writer方法， 另一个执行reader方法，wirter方法中，写操作a=1操作会在flag=true之前，如果没有volatile的保证，我们的CPU可能会重排序这些操作，可能我们的flag先执行了， 然后在执行v=1；因为我们的编译器和CPU可能会对我们非volatitle变量进行重排序优化性能，所以可能执行的顺序不是我们预期想要的。

### 如何保证有序性？

volatile是通过内存屏障实现可见性的，。

#### 写屏障

确保我们volatile之前所有写操作都同步到主内存了，同步到主内存就有了可见性了。

#### 读屏障

确保这个读操作回从主内存读到最新的数据。包括在读屏障后的所有代码也会也会从主内存读，哪怕不是volatile

### 可见性

对volatile变量写操作会立刻刷新到主内存

每次volatile读取都是从主内存读取，而不是线程的缓存



## 如果我不想从主内存读了或者写进我该怎么办

`volatile` 适合用于控制标志（flag）变量，这些变量通常用于线程间的简单状态检查和控制。

~~~java
public class VolatileExample {
    private volatile boolean flag = false;

    public void setFlagTrue() {
        flag = true;
    }

    public void doWork() {
        while (!flag) {
            // do some work
        }
        // flag is true, proceed with next step
    }
}
~~~


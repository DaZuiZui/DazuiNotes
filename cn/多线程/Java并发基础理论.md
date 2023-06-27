# Java并发基础理论

## 进程与线程

### 进程

    进程是程序的一次执行过程，是系统运行程序的基本单位，因为进程是动态的。系统运行一个程序就是一个进程从创建运行到消亡的过程。
    
    我们启动main方法其实就是启动了一个JVM进程，而main方法所在的线程就是这个线程中的一个进程也称呼为主进程。

### 线程

    线程与进程相似，但是线程是一个比进程更小的执行单位。一个进程可以在执行过程中产生多个线程，线程是多个线程层贡献进程的堆和方法区资源，但是每个线程都拥有自己的程序计数器、jvm stack和本地方法栈，一个系统产生一个线程或者在各个线程之间切换工作负担要比进程小很多。

## 线程与进程的关系

    线程是进程划分的更小的运行单位，线程和进程最大的不同在于是，进程是独立的，而各线程则不一定他们会贡献一些资源，像堆和元空间。线程执行开销小，但是不利于资源的管理和保护，而进程正好相反。

## 程序计数器为什么是私有的

    程序计数器主要有2个作用

1. 字节码解器起通过改变程序计数器一次读取指令，从而实现代码的流程的控制，如：顺序、选择、循环、异常处理。
2. 在多线程的情况下，程序计数器用于记录当前线程执行的位置。从而当线程被切换回来的时候能够知道该线程上次执行到哪里了。

   例如当一个线程因为实践篇货其他高优先级的线程来了，系统就会暂停线程的执行，并将控制权转移到另一个线程，当这个线程在被调用的时候就会根据程序计数器的值告诉他上次执行到哪里了。

       如果执行的是本natvie方法，那么程序计数器记录的就是undefined地址。

   **所以程序私有主要为了线程切换后能后恢复争取的位置执行**

## jvm stack和本地方法栈为什么是私有的

    jvm stack：每个java方法在执行之前会创建一个栈桢用于存储变量标、操作数栈、常量池饮用等信息。才能够方法调用直至执行完毕的过程，就对应一个栈桢在jvm stack一个入栈和出栈的过程。
    
    本地方法栈：和虚拟机所发挥的作用非常类似，区别是：虚拟机栈为执行java方法（字节码）服务，而本地方法栈则为虚拟机使用到的本地方法服务。在HotSpot虚拟机中和java虚拟机合二为一。

## 堆和方法区

    堆和方法区是所有线程共享的资源，其中堆是进程最大的一块内存，用来存储新创建的对象。
    
    方法区主要用于存放已被夹在的类信息、常量、静态变量、即使编译器后的代码等数据。

## 并发和并行

    并发：两个及两个以上的作业在同一时间段内执行。
    
    并行：两个及两个以上的作业在同一时刻执行。

## 同步和异步的区别

    同步：发出一个调用之后，在没有得到结果之前，该调用就不可以返回，一直等待。
    
    异步：调用在发出之后，不用等待返回结果，该调用直接return；

## 为什么使用多线程

### 从计算机底层角度来说

    线程可以比作是轻量级的线程，是程序执行的最小单位，线程间的切换和调度的成本远远小于进程。另外多核CPU时代意味着多个线程可以运行，这减少了线程上下文切换的开销。

### 从互联网发展趋势来说

    现在的系统并发要求很大， 而多线程并发编程就是高并发系统的基础，这就可以利用好多线程机制可以大大提高系统整体的并发能力和性能。

## 使用多线程可能带来什么问题

    并发编程的目的就是为了提高程序的执行效率提高程序的速度，可能带来的问题有内存泄漏、死锁、线程不安全等等。

## 如何理解线程安全和不安全

线程安全和不安群啊是对同一份数据的访问是否能达到一致性和正确性

## 线程的生命周期

Java线程的生命周期有6种状态，操作系统层面来看有7种

new：初始状态，线程创建出来，没有调用start方法

READY：可运行状态（在操作系统角度来看）

runnable：运行状态，线程调用了start等待运行的状态

blocked：堵塞状态，需要等待锁释放

waiting：等待状态，表示线程需要等待其他线程做出一些特定的动作。

time_waiting：超时等待状态，等待时间后会重新竞选CPU使用权。

terminated：终止状态，表示线程已经运行完毕。

线程的状态是随着代码的执行在不同状态之间切换的。

    当我们创建线程它属于new状态， 调用了start方法就是ready（可运行状态），可运行状态的线程获得了CPU时间片（timeslice）后就处于runnning（运行）状态，当线程执行了wait方法就会进入waiting状态，进入等待状态需要其他线程通知才能返回到运行状态。如果通过sleep和wait（long mullis）方法可以将线程状态编程timed——waiting状态，当超时时间结束就会返回runnning状态。当一个线程进入synchhronized的时候如果该方法块已经被其他线程持有锁那么就会进入堵塞状态，直到它获取到锁，当线程调用wait方法后，它会释放持久的锁并且进入等待状态，当其他线程notity这个这个线程，被唤醒的线程会尝试重新获取锁这个时候就会进入堵塞blocked状态。当鲜橙国之ing晚run方法之后就会进入terminated终止状态

## 线程的上下文切换

线程在执行过程中会有自己的运行条件和状态（也称上下文），比如上文所说过的程序计数器，栈桢信息等。

    当我们调用sleep和wait主动让出cpu的时候
    
    cpu时间片用完，因为操作系统要防止一个线程货进程长时间占用CPU导致其他线程或者进程饿死。
    
    调用了堵塞类型的系统中断，比如请求IO，线程被堵塞
    
    被终止或结束运行。

前三者上下文切换的时候会保留信息线程的上下文信息，留着下一次使用的时候恢复现场。

## 什么是死锁，如何避免死锁

线程死锁就是多个线程被堵塞，他们中的一个活着全部都在等待某个资源释放，由于线程被无限期的堵塞，因此程序不可以正常结束。

```java
public class DeadLockDemo {
    private static Object resource1 = new Object();//资源 1
    private static Object resource2 = new Object();//资源 2

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (resource1) {
                System.out.println(Thread.currentThread() + "get resource1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource2");
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                }
            }
        }, "线程 1").start();

        new Thread(() -> {
            synchronized (resource2) {
                System.out.println(Thread.currentThread() + "get resource2");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource1");
                synchronized (resource1) {
                    System.out.println(Thread.currentThread() + "get resource1");
                }
            }
        }, "线程 2").start();
    }
}

```

## 如何预防死锁

    我们可以通过一次性申请所有资源和占用部分线程资源的线程如果去申请其他线程如果申请不到就主动释放它占用的资源和按照一定的顺序去申请资源，释放的时候就反序列释放

## 如果避免死锁

    避免死锁就是在资源分配的时候借助算法如银行家算法对资源进行评估、使线程进入安全状态，如P1 P2 P3

```java
new Thread(() -> {
            synchronized (resource1) {
                System.out.println(Thread.currentThread() + "get resource1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource2");
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                }
            }
        }, "线程 2").start();

```

## sleep方法和wait方法的区别

他们的共同点都可以暂停线程的执行。

sleep方法没有释放锁，而wait释放了锁

wait通常用于线程间的交互 or 通信，sleep通常用来暂停执行。

wait方法被调用了不会自动苏醒，需要别的线程notify或者notifuAll，sleep方法执行完毕后会自动苏醒，但是我们也可以给wait方法传递指定的时间，在指定的时间后也会苏醒。

sleep是thread类的静态本地方法，wait则是Object类的本地方法。

## 为什么wait方法不定义在Thread

wait方法是获得对象锁的线程实现等待，会自动释放当前线程占用的对象锁，每个对象都拥有对象锁，释放锁让让对象锁进入waiting状态，自然就是要操作对象的Object而非Thead

## 为什么sleep不定义在Object中

因为sleep就是让当前线程暂停执行，不涉及对象类，也不需要获得对象锁。

## 可以直接调用Thread类的run方法执行吗

答案是可以的，如果这么做只会当做一个main线程下的普通方法去执行，并不会在某个线程中执行，因为new Thread的时候线程就进入了new状态，调用start方法就进入了就绪状态，分配到时间片就开始运行，start会执行线程会执行相应的准备工作，然后自动执行run（）方法，这就是真正的多线程工作。


参考文献Javaguide

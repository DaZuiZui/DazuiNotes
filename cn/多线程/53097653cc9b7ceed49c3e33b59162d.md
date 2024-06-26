#  Runnable和Thread区别

Thread实现了Runnable接口，而且内部还有一个Thread状态的管理机制（线程的生命周期）。

## Thread

当我们调用start方法的时候，底层调用了本地方法，让我们的操作系统的的线程库来启动一个新的本地线程。

我们执行start方法的时候其实执行的就是Thread的run方法

当然Thread也有一个很明显的缺点。我们的Java不支持多继承，也就是说我们类继承了Thread就不能继承其他的类了，限制了我们类的扩展性。

~~~java
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread is running");
    }
}

public class Main {
    public static void main(String[] args) {
        MyThread t1 = new MyThread();
        t1.start(); // 启动线程
    }
}
~~~



## Runnable接口

Runnable是一个功能接口（只有一个方法的接口），他没有任何线程管理能力，只定义了一个任务，Thread就是使用Runnable对象执行任务。

如果我们把Runnable传递给Thread的构造函数，Thread调用start方法的时候，就是执行的我们Runnable的run方法。Runnable就是定义一个线程任务。

~~~java
class MyRunnable implements Runnable {
    public void run() {
        System.out.println("Runnable is running");
    }
}

public class Main {
    public static void main(String[] args) {
        MyRunnable myRunnable = new MyRunnable();
        Thread t1 = new Thread(myRunnable);
        t1.start(); // 启动线程
    }
}
~~~

## `Thread` 和 `Runnable` 底层实现有何不同？

Thread是调用本地方法，让我们的操作系统启动一个新的线程

Runnable是定义一个线程任务，然后让我们的Thread执行或者我们的线程池。
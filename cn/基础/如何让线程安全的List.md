# 如何让线程安全的List

## Collections.synchronizedList

首先是用Collections.synchronizedList方法可以把普通的List转为线程的安全List。所有对该List的访问都会被同步

~~~java
import java.util.Collections;
import java.util.List;
import java.util.ArrayList;

public class ThreadSafeListExample {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        List<String> synchronizedList = Collections.synchronizedList(list);

        // 创建多个线程来访问synchronizedList
        Runnable task = () -> {
            for (int i = 0; i < 10; i++) {
                synchronizedList.add(Thread.currentThread().getName() + " - " + i);
            }
        };

        Thread thread1 = new Thread(task, "Thread 1");
        Thread thread2 = new Thread(task, "Thread 2");

        thread1.start();
        thread2.start();

        try {
            thread1.join();
            thread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // 输出列表内容
        synchronized (synchronizedList) {
            for (String item : synchronizedList) {
                System.out.println(item);
            }
        }
    }
}

~~~

但是一定要注意，循环的时候一定要用synchronized，因为如果我们在进行循环print的时候，别人线程增加了一个删除了一个会有问题。

synchronizedList内部是通过synchronized实现的。

## CopyOnwriteArrayList

### 写操作

每次写操作的时候都会创建一个副本，然后在这个副本进行操作，然后拿这个副本代替正式的数据。这个操作保证同一时刻只有一个线程在执行。

## 读操作

读操作直接去读底层真实的数据而不是副本。
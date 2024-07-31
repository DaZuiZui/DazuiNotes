# AQS

AQS是Java并发包的一个框架，用于实现同步器（如锁和其他同步器），他主要是基于FIFO(先进先出)等待队列的机制来实现线程的同步和协调。

## AQS核心概念

### 同步状态（state）

AQS通过一个volatile的int变量的state啦i表示同步状态。

getState（），setState()

### 队列

AQS使用一个FIFO等待队列来管理被堵塞的线程。这个队列是一个双向链表，每个节点（Node）代表一个被堵塞的线程。节点包含了线程的引用和等待状态等信息。

### 独占模式和共享模式

AQS支持两种模式：独占模式（Exclusive）和共享模式（Shared）。独占模式下只有一个线程能持有同步锁（如ReentranLock）；在共享模式下，多个线程可以共享同步状态（如CountDownLatch）。

## 主要方法

### 获取锁

**acquire(int arg)和acquireShared(int arg)**

这两个方法用于获取锁或同步状态。acquire用于独占锁模式，acquireShared用于共享模式。主要是用来获取锁，如果获取失败就将现场加入等待队列并堵塞。

### 释放锁

**release（int arg）和releaseShare（int arg）**

这两个方法用于释放锁或同步状态，如果释放锁成功就唤醒等待队列中的线程。

### 获取同步状态

**tryAcquire（int arg）和tryAcquireShared（int arg）**

这两个方法由子类实现，决定了线程是否能成功获取同步状态。

### 释放同步状态

tryReplease(int arg)和tryRelesaseShare（int arg）

 决定线程是否能释放同步状态。


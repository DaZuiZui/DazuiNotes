#  ThreadLocal底层分析

ThreadLocal可以实现【资源对象】的线程隔离，让每个线程各用各的【资源（变量）】，避免征用引发的线程安全问题了。

起到了隔离作用是每个线程中的Map，threadLocal是关联每个资源对象（变量），因为Key是线程ThreadLocal的实列。

总结出：**线程之间可以实现资源隔离，线程内是可以实现资源贡献的**



我们通过这个代码可以得知线程之间是有隔离性的

~~~java
public class Utils {
    private static final ThreadLocal<User> t = new ThreadLocal<>();

    public static User getUser(){
        User u =  t.get();
        if (u == null){
            u = new User();
            t.set(u);
        }

        return u;
    }
}

class Test1{
    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            new Thread(() ->{
                System.out.println(Utils.getUser());
            }).start();
        }
    }
}
~~~

结果：

~~~java
0 is com.example.springtest.User@48e6a8
3 is com.example.springtest.User@673956
2 is com.example.springtest.User@1a22f1d
1 is com.example.springtest.User@14827d5
4 is com.example.springtest.User@106a70b
~~~





我们通过这个代码可以得知线程内是实现了资源共享

```java
public class Utils {
    private static final ThreadLocal<User> t = new ThreadLocal<>();

    public static User getUser(){
        User u =  t.get();
        if (u == null){
            u = new User();
            t.set(u);
        }

        return u;
    }
}

class Test1{
    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            int finalI = i;
            new Thread(() ->{
                System.out.println(finalI +" is "+Utils.getUser());
                System.out.println(finalI +" is "+Utils.getUser());
                System.out.println(finalI +" is "+Utils.getUser());
            }).start();
        }
    }
}
```

结果

~~~java
0 is com.example.springtest.User@5802d8
0 is com.example.springtest.User@5802d8
0 is com.example.springtest.User@5802d8
3 is com.example.springtest.User@1a22f1d
4 is com.example.springtest.User@673956
1 is com.example.springtest.User@48e6a8
1 is com.example.springtest.User@48e6a8
2 is com.example.springtest.User@14827d5
1 is com.example.springtest.User@48e6a8
4 is com.example.springtest.User@673956
3 is com.example.springtest.User@1a22f1d
4 is com.example.springtest.User@673956
2 is com.example.springtest.User@14827d5
3 is com.example.springtest.User@1a22f1d
2 is com.example.springtest.User@14827d5

Process finished with exit code 0

~~~

## ThreadLocal原理

**ThreadLocal的原理就是每一个线程之间都有ThreadLocalMap类型的成员变量，用来存储资源。**

	1.调用set方法就是以ThreadLocal自己作为key，对象资源作为value,放入当前线程的ThreadLocalMap集合中，
	
	2.调用get方法就是以ThreadLocal自己作为key，到当前线程中查找相关的资源值
	
	3.调用remove方法就是以ThreadLocal自己作为key，移除当前线程关联的资源值

 


## ThreadLocal的基本使用

| 方法                     | 描述                                                   |
| ------------------------ | ------------------------------------------------------ |
| ThreadLocal()            | 创建ThreadLocal对象                                    |
| public void set(T value) | 设置当前线程绑定的局部变量    //将变量绑定到当前线程中 |
| public T get()           | 获取当前线程绑定的局部变量    //获取当前线程绑定的变量 |
| public void remove()     | 移除当前线程的局部变量                                 |

## ThreadLocal与Synchronized的区别

	ThreadLocal与synchronized关键字都是用于处理多线程并发访问变量的问题，不过两者的处理问题的角度和思维不一样，

| Synchrinized                                                 | ThreadLocal                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| synchronized采用时间换空间的方式，只提供了一份变量，让不同的线程排队访问 | ThreadLocal采用空间换时间的方式，为每个线程都提供了一份变量副本，从而实现线程同时访问互相不受干扰 |
| 多线程之间访问资源的同步（线程之间要排队）                   | 多线程中让每个线程之间的数据互相隔离                         |

所以使用ThreadLocal有更高的并发性

## ThreadLocal的内部结构

	在JDK8中ThreadLocal的设计方式是，每一个Thread维护一个ThreadLocalMap，这个Map的Key是ThreadLocal实例本身，value是要存储的值
	
	具体的过程是这样的
	
		1.每一个thread线程内部都有一个Map（ThreadLocalMap）
	
		2.Map里面存储ThreadLocal对象（Key）和线程的变量副本（value）
	
		3.Thread内部的Map是由ThreadLocal维护的，由ThreadLocal负责向map获取和设计线程的变量值
	
		4.对于不同的线程，每次获取副本值时，别的线程并不能获取到当前线程的副本值，形成了副本的隔离



这么设计的好事是每个Map存储的Entry数量变少，2.当Thread销毁的时候ThreadLocalMap也会跟着销毁，减少内存的使用

##  ThreadLocal核心方法源码

threadLocal对外暴露的方法有4种

| 方法                       | 描述                         |
| -------------------------- | ---------------------------- |
| protected T initialValue() | 返回单枪线程局部变量的初始值 |
| public void set(T value)   | 设置当前线程绑定的局部变量   |
| public T get()             | 获取当前线程绑定的局部变量   |
| public void remove()       | 移除当前线程绑定的局部变量   |



### set方法源码

```java
/*
 * 设置当前线程对应的ThreadLocal的值
 */
public void set(T value) {
    //获取当前的线程对象
    Thread t = Thread.currentThread();
    //获取次线程对象种维护的ThreadLocalMap对象
    ThreadLocalMap map = getMap(t);
    //判断该map是否存在
    if (map != null)
        //如果存在就调用该map去设置此实体Entry
        map.set(this, value);
    else
        //如果线程Thread不存在ThreadLocalMap对象
        //就调用creatMap进行ThreadLocalMap对象初始化
        //并将t(当前线程)和vlaue(t对应的值)作作第一个entry存放至ThreadLocalMap中
        createMap(t, value);
}

ThreadLocalMap getMap(Thread t) {
	//t就是当前线程，threadLocal就是当前线程对应维护的ThreadLocal
    return t.threadLocals;
}

void createMap(Thread t, T firstValue) {
     //这个this是调用此方法的threadLocal，firstValue就是要插入的值
     t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

通过上面代码我们得知，set方法主要做了

	1.先获取ThreadLocalMap通过当前线程
	
	2.如果获取结果为null，就给该线程创建threadlocalmapmap，并将数据设置到ThreadLocal中
	
	3.如果不为null，就将参数设置到Map数组中



### get方法

```java
public T get() {
    //获取当前线程对象
    Thread t = Thread.currentThread();
    //获取此线程中维护的ThreadLocalMap
    ThreadLocalMap map = getMap(t);
	//判断是否存在
    if (map != null) {
        //以当前的threadLocal为key调用geentry获取对应存储的实例
        ThreadLocalMap.Entry e = map.getEntry(this);
        //查看实例不为null就返回value
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    /*
     *	如果执行初始化就有2种可能
     *		第一种情况：map不存在，表示此线程没有维护的ThreadLocalMap对象
     *		第二种情况: map存在，但是没有与当前ThreadLocal关联的Entry(实体)
     *		此方法的作用就是返回当前ThreadLocal的初始值
     */
    return setInitialValue();
}

  private T setInitialValue() {
      	//
        T value = initialValue();  
       Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            //设置实体
            map.set(this, value);
        else
            //如果map为null 就创建map对象
            createMap(t, value);
        return value;
 }
```

通过上面代码我们可以知道：

	1.首先获取当前线程，根据线程获取一个map
	
	2.如果获取的map不为null，就在Map种以ThreadLocal的引用作为key来在Map中获取对应的Entry，如果获取到了就返回它的值，如果没有获取到就通过initialVlue函数获取初始化value（默认为null），然后用 threadLocal的引用和val作为key和value创建一个新的map

**总结一句话：先获取线程ThreadLocalMap变量，如果存在就返回值，不存在就创建并返回初始值**

### remove方法

```java
public void remove() {
    //获取当前线程对象中维护的ThreadLocalMap对象
    ThreadLocalMap m = getMap(Thread.currentThread());
    //如果存在
    if (m != null)
        //就调用map.remove进行删除  通过ThreadLocal的引用为key来删除
        m.remove(this);
}
```

通过上面代码我们可以得知，

	1.首先根据当前线程获取ThreadLocalMap
	
	2.如果获取的Map不为空就移除当前ThreadLocal的引用为key对应的实体就好了

## ThreadLocalMap源码分析

	ThreadLocalMap是ThreadLocal的内部静态类，它没有实现Map接口，但是它用独立的方式实现了Map的功能，其内部的Entry也是独立实现的。

```JAVA
/*
 * 初始容量 必须是2的n次幂
 * 是2的n次幂就是为了降低hash冲突 
 * 详细的解读在下面的TreadLocal解决Hash冲突问题
 */
private static final int INITIAL_CAPACITY = 16;

/**
 * 我们所说的KV
 */
private Entry[] table;

/**
 * 数组中元素的个数，用来判断是否超越阈值
 */
private int size = 0;

/**
 * 阈值
 */
private int threshold; // Default to 0
```

##  ThreadLocal解决Hash冲突

~~~java
/*
 * 设置当前线程对应的ThreadLocal的值
 */
public void set(T value) {
    //获取当前的线程对象
    Thread t = Thread.currentThread();
    //获取次线程对象种维护的ThreadLocalMap对象
    ThreadLocalMap map = getMap(t);
    //判断该map是否存在
    if (map != null)
        //如果存在就调用该map去设置此实体Entry
        map.set(this, value);
    else
        //如果线程Thread不存在ThreadLocalMap对象
        //就调用creatMap进行ThreadLocalMap对象初始化
        //并将t(当前线程)和vlaue(t对应的值)作作第一个entry存放至ThreadLocalMap中
        createMap(t, value);
}

ThreadLocalMap getMap(Thread t) {
	//t就是当前线程，threadLocal就是当前线程对应维护的ThreadLocal
    return t.threadLocals;
}

void createMap(Thread t, T firstValue) {
     //这个this是调用此方法的threadLocal，firstValue就是要插入的值
     t.threadLocals = new ThreadLocalMap(this, firstValue);
}
~~~

首先看一下ThreadLocalMap构造方法

```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    //初始化table
    table = new Entry[INITIAL_CAPACITY];
    //计算索引
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    //设置值
    table[i] = new Entry(firstKey, firstValue);
    //设置元素个数
    size = 1;
    //设置阈值
    setThreshold(INITIAL_CAPACITY);
}
```

当达到了当前数组容量的2/3就会进行扩容，扩容2倍



**再来看下set(ThreadLocal<?> key, Object value)方法**

```java
/*
 * 
 */
private void set(ThreadLocal<?> key, Object value) {

    // We don't use a fast path as with get() because it is at
    // least as common to use set() to create new entries as
    // it is to replace existing ones, in which case, a fast
    // path would fail more often than not.

    Entry[] tab = table;
    int len = tab.length;
    //指定下标计算
    int i = key.threadLocalHashCode & (len-1);

    for (Entry e = tab[i];
         e != null;
         //nextIndex是获取环形数组的下一个节点
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
		//如果key值存在直接替换value
        if (k == key) {
            e.value = value;
            return;
        }
		
        //key为null，但是值不为null，说明之前的ThreadLocal对象已经被回收了
        //当前数组中的Entry是一个陈旧的元素
        if (k == null) {
            //用心的元素替换旧元素，这个方法做了不少的垃圾清理动作，防止内存泄漏
            replaceStaleEntry(key, value, i);
            return;
        }
    }
	//ThreadLocal对应的key不存在并且没有找到旧的元素，则在空元素的位置创建一个新的Entry
    tab[i] = new Entry(key, value);
    int sz = ++size;
    //cleanSomeSlots用于清理那些e.get（）为null的元素
	//如果没有清理任何entry，并且当前使用的量达到了阈值，那么久进行rehash(执行一次全表的扫描清理工作)
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```

我们通过这些代码可以得出，添加方法就是

	**1.根据key计算出索引，然后查找i位置上的Entry**
	
	**2.如果Entry已经存在，并且存在的key跟要传入的key一样，那么这时候就给这个key复制新的val值**
	
	**3.如果entry存在，但是key为null，那么就调用replaceStaleEntry来更换这个key为null的Entry**
	
	**4.不断循环检测，直到遇见为null的地方，这时候要是没有在循环过程中return，那么久在这个null的位置新建一个Entry然后并擦汗如数据，同时size++；**
	
	**最后调用cleanSomeslots,清理Key为null的Entry，然后返回是否清理了Entry，接下来判断siez是否>=阈值达到了rehash条件，如果达到了就会调用rehash进行执行一次全表扫面清理。**



**ThreadLocalMap使用线性探测法来解决Hash冲突的，该探测法就是一直探索下一个地址，直到找到有空的地址后插入，如果找不到空余的地址就会产生溢出**

 for example	

	如果一个table的长度是16，然后计算出来的hash值是14，如果当前下标已经有数据了，key又不相等，那么就判断15的位置，如果15也不满足要求就看0 ...... 直到找到一个下标为null的时候插入进入然后停止，如果一直找不到位置就会产生溢出。



## ThreadLocal 内存泄漏

### 强引用

	强引用就是我们常见的普通对象引用，只要还有洽购引用只想一个对象，就能表明对象还或者，垃圾回收器就不会回收这个对象

### 弱引用

	垃圾回收器一旦发现了只有具有弱引用的对象，不管当前内存空间是否足够，都会回收它的内存

**无论TreadLocal使用了强引用还是弱引用都无法避免内存泄漏，因为造成ThreadLocal的内存泄漏的根本原因就是ThreadLocalMap的生命周期和Thread一样长，如果没有手动删除对应的Key就会造成内存泄漏，我们只能通过这两办法去避免内存泄漏，1.使用完ThreadLocal就调用remove删除对应的Entry，2.使用完ThreadLocal，让当前的Thread耶跟着运行结束**

## 总结

### ThreadLocal的作用

	ThreadLocal实现了资源对象的线程隔离,县城内的资源共享

### ThreadLocal的原理

	ThreadLocal的原理就是让每一个Thread都有一个ThreadLocalMap进行存储资源，
	
			调用set方法就是当前ThreadLocal的引用作为Key，资源作为Value注入进入。
	
	 	  调用get方法就是当前ThreadLocal的引用作为Key，获取对应的资源
	
			调用remvoe方法就是当前ThreadLocal的引用作为key，移除对应的资源

### set方法的执行流程

	1.获取一个ThreadlocalMap通过当前线程
	
	2.如果获取的结果为null就给该线程创建map并设置要插入的数据
	
	3.如果不为null，就将参数设置到Map中(Key为ThreadLocal的引用)

### get方法的流程

	1.通过当前线程去获取ThreadLocalMap
	
	2.如果获取的ThreadLocalMap不为null，把ThreadLocal的引用作为Key，如果查找到了就返回val，如果没有查找到就调用initialValue获取创建初始化的值，然后用ThreadLocal的引用和Vlaue创建一个新的Map

### remove方法

	1.通过线程去获取ThreadLocalMap
	
	2.如果获取的ThreadLocalMap不为null，就移除Key为ThreadLocal的引用对应的实例(Entry)就好了

### ThreadLocalMap内部结构

	**ThreadLocalMap**是**ThreadLoacl**的内部类，它没有实现Map的接口,但是ThreadLocalMap用了独特方的方法实现了Map，其中的Entry也是。

### ThreadLocalMap解决Hash冲突

	ThreadLocal是通过**线性探测法**去解决Hash冲突的，该探测法就是一直探索下一个地址，探索到为null的地址就插入，如果一直找不到地址为null的地方就会产生溢出

### ThreadLocal内存泄漏

	ThreadLocal无论使用强引用还是弱引用都无法避免内存泄漏，因为ThreadLocal的内存泄漏根本原因就是ThreadLocalMap的生命周期是和Thread一样的，如果没有手动删除对应的Key就会造成内存泄漏，我们只有2种办法去解决1.使用完ThreadLocal就去删除它对应Entrey，Threadlocal使用完，Thread也跟着删除。

### ThreadLocalMap扩容

​	当ThreadLocalMap中的Entry数量达到了数组2/3，并且没有清理到任何数据就会扩容。

​	扩容会先清理工作，对过期不在引用的Entry进行清理，清理完成后如果没有清理到任何Entry说明THreadLocalMap中的数据几乎都过期了，可以扩容操作，扩容2倍，扩容后会重新计算Entry所在的位置，如果发生冲突就线性探索法。扩容完毕后新的数组代替旧的数组。

### Synchronized与ThreadLocal的区别

Synchronized和ThreadLocal都是用来解决多线程并发访问变量的问题，只不过他俩解决的方式的思维不一样

Synchronized是以时间换空间，让线程排队去访问资源，ThreadLocal是以空间换时间，为每个线程都提供了一份变量副本，从而实现多线程中互不受干扰。

ThreadLocal不是用来解决资源共享的，Threadlocal确实可以解决多线程下的线程安全问题，但是资源并不是共享的，而是每个线程独享的。

## ThreadLocal的使用场景

1.在进行对象跨层传递的时候，使用ThreadLocal可以避免多次传递，打破层次间的约束

		比如说我每次要传一个参数的时候都要在方法上加一个形参比较麻烦，这时候就可以用ThreadLOcal去解决 

2.线程间的数据隔离

3.进行事务操作，将事务存储到线程信息里面

4.数据连接,Session会话管理

	**比如说Spring框架在事务开始的时候会给当前线程绑定一个JDBC connection，在整个事务过程都是使用该线程绑定的connection来执行数据库操作的，实现了事务的隔离性，Spring框架里面就是用ThreadLocal来实现这种隔离的。**

## ThreadLocal下标查找

​	通过ThreadLocal通过他的hashcode计算出索引下标，然后对应的Entry，key是ThreadLocal，value是值。

### 我对ThreadLocal的理解

	ThreadLocal实现了线程之间的资源隔离，线程之内的资源共享，做到隔离作用的是每个Thread的ThreadLocalMap，ThreadLocal是关联每一个资源，它的原理就是每一个Thread都有一个ThreadLocalMap进行存储资源，set方法就是ThreadLocal的引用作为key，资源作为value进行注入进入，get方法就是ThreadLocal的引用作为key去获取value，remove就是ThreadLocal的引用作为Key对应的实例。ThreadLocalMap是ThreadLocal的内部类，ThreadLocalMap没有实现Map接口，但是它以独特的方式实现了Map的功能，其内部的Entry也是，ThreadLocal解决Hash冲突是用的线性探测法去解决的，ThreadLocal的扩容机制是当数组里面的Entry数量到了数组长度的2/3就会执行rehash方法，rehash首先会探测式清理工作，从table的起始位往后清理，清理完成后可能有一些key为null的数据会被清理掉，然后在判断size是否大于 阈值 * 3/4 （**`threshold - threshold`**）如果大于了就扩容，去调用resize方法，reseize方法会遍历老的数组，将里面的数组重新计算的新下标放入到新的数组，如果发生了hash冲突就用先行探索法去解决，ThreadLocal内存泄漏的原始是因为ThreadLocalMap的生命周期和Thread一样，如果没有手动删除对应的key就会造成内存泄漏，我们只有2种办法去解决内存泄漏，1使用完threadlocal就删除对应他的key   2.threadlocal使用完，thread也跟着删除，它的使用场景比如说，在Spring框架中，在事务的开始会给当前线程绑定一个JDBC connection，在整个事务过程中都是使用该线程绑定的Connection来执行数据库操作的，实现了事务的隔离性，Spring框架里面就是用了ThreadLocal来实现这种隔离的。
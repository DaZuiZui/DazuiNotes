# 线程安全的list

## CopyOnWriteArrayList

这个是一个线程安全的list，他的思想就是写的时候复制一份（add，remove，update），他的思想就是在新的数组修改，然后替换之前的数组。适合写少读多

### 内部实现

他使用了volatitle，保证了我们读的时候数据是从主内存拿的，写的时候是直接写入主内存。也保证了读之前的执行，和阻止了cpu重新排序的症状。

~~~java
private transient volatile Object[] array;
~~~

### read

读操作直接在当前数组进行操作

~~~java
public E get(int index) {
    return get(getArray(), index);
}

final Object[] getArray() {
    return array;
}
~~~

### 写操作

保证了同一时间只有一个线程在操作

~~~java
public E set(int index, E element) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        E oldValue = get(elements, index);

        if (oldValue != element) {
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len);
            newElements[index] = element;
            setArray(newElements);
        }
        return oldValue;
    } finally {
        lock.unlock();
    }
}

~~~

## vector

这个就是对所有写的方法加了synchronized。特点是开销比较稳定

## **Collections.synchronizedList(List<T> list)**：

和vector一样，所有都加了synchronzied
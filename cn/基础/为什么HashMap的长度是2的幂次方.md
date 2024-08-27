# 为什么HashMap的长度是2的幂次方

## 取模运算简化为位运算

如果我们确定存储位置需要

~~~java
index = hash % array.length;
~~~

如果我们是2的幂次方可以简化为

~~~java
index = hash & (array.length - 1);
~~~

## hash均匀分布

HashMap是根据key来确定数组的下标的，如果n为2幂次方就可以保证数据均匀的插入，如果不是2的幂次方，可能数组的一些位置永远也不会插入数据，这样就浪费了数组的空间还加大了hash冲突，通过%来确定位置效率远远不如&运算，但是如果使用&的话必须要保证length（数组长度）是2的n次幂。HashMap使用2的n次幂放就是为了保证数据均匀分布减少hash冲突，如果hash冲突越大，代表数组中的一个链或者红黑树长度越大，这样会降低hashmap的性能，

## 扩容效率提升

在HashMap扩容通常是容量扩大为当前容量2的2倍，新数组的长度依然是2的幂次方，这样可以保证旧数组中的数据和新数组的数据要么位置相同，要不就是旧位置+旧数组长度位置。 这样使得我们分散更加高效。

## 如果我们指定初始容量不是2的幂次方会发生的事情

如果给的值不是2的幂次方，那么就通过底层右移操作，变成大于你指定的数字最小的二次幂

我们传入的数组长度大小不能超过2^30

~~~java
public HashMap(int initialCapacity, float loadFactor) {
    //如果我们给的长度小于0，就抛出异常
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    //如果我们给的长度大于2^30 那么就把2^30次方赋值给它，意思就是我们的长度必须小于2^30
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    //设置数组长度
    this.threshold = tableSizeFor(initialCapacity);
}
~~~

~~~java
/**
 * Returns a power of two size for the given target capacity.
 * 或运算都是0的时候为0有1的时候为1
 * 假设cap传入的是10
 */
static final int tableSizeFor(int cap) {
    //这里-1是为了防止出现我们传入了16 还进行了右移运算，假如我们输入成16 通过-1然后最后右移算出来是16
    int n = cap - 1;		 
    /*
    	00001001	9
    	00000100	4	9 >>> 1
    	--------------
    	00001101	13	9 |= 4
    */
    n |= n >>> 1;		
    /*
    	00001101	13
    	00000011	3
    	---------------
    	00001111	15
    */
    n |= n >>> 2;
    /*
     	00001111	15
     	00000000	0
     	--------------
     	00001111	15
     */
    n |= n >>> 4;
       /*
     	00001111	15
     	00000000	0
     	--------------
     	00001111	15
     */
    n |= n >>> 8;
       /*
     	00001111	15
     	00000000	0
     	--------------
     	00001111	15
     */
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
~~~


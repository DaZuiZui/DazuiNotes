# HashMap源码分析

## HashMap的数据结构

### Hash table（Hash 表）

HashMap的核心数据结构是一个Hash表，它实际上是一个数组，每个元素都被称呼为一个桶（Bucket）。这个数组存储了所有的K—V。

### 桶(Bucket)

每个桶都是存储单元，通常是一个链表或者一个红黑树，每个桶可以存储多个K-V对，桶的数量由HashMap的容量决定。

### 链表和红黑树（LinkedList or red black tree）

​	**链表：** 如果发生Hash冲突，就会使用拉链法去解决，就是添加到链表尾部。

​	**红黑树：**当链表长度大于等于阈值（默认是8）就会转换为红黑树。红黑树是一种自平衡的二叉搜索树，可以在对数时间内完成查找操作。红黑树的诞生就是解决链表过长查找时间过长的问题。

### 键值对

​	每个桶中存储的就是K-V对，通过key去找到Value。hash table是通过key的hashcode来确定存储位置，并在通过比较key来找到对应的value。

### hash函数

​	hashtable使用hash函数将key的hashcode映射到索引的位置，hash函数的目的就是将key均匀的分散到各个桶中，减少hash冲突。

### 扩容

​	当hashtable的key-value数量到达一定百分比（负载因子）的时候，hashMap会自动扩容，以增加容量并重新分配到新的桶中。保证了HashMap的性能。

## HashMap的扩容机制	

HashMap的扩容机制是通过resize方法实现的。

​		1.在我们插入k-v的完成之后，首先检查是否需要扩容，如果当前的k-v对大于我们的负载因此百分比（默认是0.75 百分之75），那么就会触发扩容机制。

​		2.在resize方法中扩容第一步首先是计算新的容量，通常是之前容量的2倍，新的容量必须是2的幂次方，然后创建一个新的hash table，将就的k-v重新分配到新的桶中。这个重新分配的过程需要重新计算hashcode确定新的位置。

​		3.在迁移过程如果发生了冲突，那么就会按照链表或者红黑树的方法去解决。

​		4.完成扩容，如果已经完成了扩容旧的hashtable会被丢弃，新的hashtable会替换为新的hash表，而重新计算阈值以便下一次扩容。

~~~java
/**
 * 扩容哈希表
 * @param newCapacity 新的容量
 */
void resize(int newCapacity) {
    // 保存旧的哈希表
    Node<K,V>[] oldTable = table;
    // 获取旧哈希表的容量
    int oldCapacity = (oldTable == null) ? 0 : oldTable.length;
    
    // 如果旧哈希表为空，表示是初始化
    if (oldCapacity > 0) {
        // 如果已经达到最大容量，不再扩容
        if (oldCapacity >= MAXIMUM_CAPACITY)
            return;
        
        // 创建新哈希表
        Node<K,V>[] newTable = new Node[newCapacity];
        
        // 迁移数据到新哈希表
        for (int i = 0; i < oldCapacity; i++) {
            Node<K,V> e;
            if ((e = oldTable[i]) != null) {
                oldTable[i] = null;  // 释放旧哈希表的引用
                if (e.next == null)  // 该桶中只有一个键值对
                    newTable[e.hash & (newCapacity - 1)] = e;
                else if (e instanceof TreeNode)  // 红黑树节点
                    // 在新哈希表中保持红黑树结构
                    ((TreeNode<K,V>)e).split(this, newTable, i, oldCapacity);
                else {  // 链表节点
                    // 将链表中的键值对分散到新的桶中
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCapacity) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        } else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTable[i] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTable[i + oldCapacity] = hiHead;
                    }
                }
            }
        }
        // 设置新的哈希表和阈值
        table = newTable;
        threshold = (int)(newCapacity * loadFactor);
    }
}
~~~

## 为什么容量比如是2的幂次方？

​	首先在hash码计算方面，HashMap使用位运算进行计算出hash码的新位置，具体来说，它会使用(n-1) & hash的方式来将hash码映射到的桶的索引，其中n是hash表的容量，hash是key的hashcode。这种计算方式必须要求n的幂次方，因为(n-1)在二进制中表达所有位都是1，也就是说只有n是2的幂次方，位运行（n-1) &hash 才能均匀的分散到不同的桶中。

​	在散列冲突解决方法：当两个不同的key有相投的hash码的时候，HashMap使用链表和红黑树来存储这些k-v，如果容量n不是2的幂次方，那么进行链表或红黑树插入和查找的时候，会 涉及到模运算，这可能导致性能下降。如果容量n是2的幂次方，模运算等操作可以通过位运算来实现，效率更高。

## put方法

​	 1.计算key的hash码（key.hashcode）然后确认插入位置。

​	2.根据
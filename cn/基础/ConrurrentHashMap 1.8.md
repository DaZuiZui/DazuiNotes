# ConrurrentHashMap 1.8

在ConrurrectHashMap内部有16个segment，每个segment都可以看作一个独立的Hashmap。

## 插入

首先根据我们的hashcode所对应的segment，然后在这个segment找到对应的桶，通过CAS算法进行插入（预期值和当前值对比和版本号）。

## 扩容

如果A segment扩容不会影响其他segment工作的， 因为他们都是独立的Hashmap。就算A segment正在扩容也不会影响A的查询，因为他有2个segment ，一个old segment一个 new segment 。



这种设计使得COncurrecthashmap能够在高并发的环境保持高效的性能，保证了线程安全。
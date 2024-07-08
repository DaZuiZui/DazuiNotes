# Mysql BufferPool和Change Buffer是如何优化读写速度的

##  BufferPool

BufferPool主要缓存最近使用的数据页和索引页，他的主要作用就是减少对硬盘的I/O操作。

BufferPool缓存了我们经常访问的页，当我们查找数据的时候会先去BufferPool中去找，找不到才会去磁盘IO。找到了就放到LRU里的老年代中，如果该页被多次使用就会提升到新生代。

然后我们数据被修改的时候，修改后的数据页会先放到Buffer Pool中，当合适的时机（Buffer pool需要 腾出空间，或者事物提交的时候）写回磁盘，这种机制叫**延迟写**

接下来就是**预读机制**了，**预读机制**就是将需要的数据页夹在到Buffer Pool。

## ChangeBuffer

如果我们现在要更新非唯一索引所在的页（page-100），并且这个page-100也不再Buffer Pool，这时候InnoDB也不会先把数据加载到内存，而是将要修改的操作记录在ChangeBuffer中，当然Changebuffer也会把内容同步给RedoLogBuffer。

然后这个时候突然来个查询也要查询page100，这时候就要把磁盘的数据加载到BufferPool，然后这个时候ChangeBuffer就会根据操作数据纠正查询出来的正确数据，然后提供给客户端，然后写入硬盘。

也就是把原本2次io变成了一次。
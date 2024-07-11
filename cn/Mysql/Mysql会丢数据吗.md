# Mysql会丢数据吗

那要看我们innodb_flush_log_At_trx_commit的值了

## 实时写，实时刷

~~~properties
inndodb_flush_log_at_trx_commit=2
~~~

这个时候修改的工作流程是先把硬盘的数据读到缓存，然后把旧的值写给undolog，然后更新缓存的数据，然后把写的这个操作给redologBuffer，然后进行commit，commit会把数据先写到pageChache。然后这时候我又修改了一条数据还是刚才所说的同步操作。pageCache会每个一秒调用一次同步函数写到redolog中。然后把BufferPool中修改的数据写到硬盘中，然后写到binlog上。

**场景A：**这种情况如果我们的数据已经写到redolog然后BufferPool挂了，这时候就不会丢数据，因为数据库重启innodb会根据undolog恢复数据。

**场景B：** BufferPool和PageCache都挂了，然后还没把数据写到redolog，这时候就会丢数据。

## 实时写实时刷

~~~yml
innodb_flush_log_at_trx_commit = 1 
~~~

这个和上面唯一区别就是当数据写到硬盘上才认为是commit，所以他不会丢数据。

## 延迟写

~~~pro
innodb_flush_log_at_trx_commit = 1 
~~~

他的区别就是当我们commit的时候，我们更新内容在redologbuffer，然后每个1秒同步一批数据到到磁盘。

如果bufferpool和redologBuffer都崩溃了那么就丢失数据了



## binlog

相当于sql二进制文件，记录了我们的执行sql的记录。可以反向实现数据恢复，或者主从。

## Redolog

记录了我修改数据块的某个数据修改什么样子。主要解决系统故障，重启服务器的时候进行恢复数据。

**主要用于崩溃恢复数据**

## Undolog

修改操作的时候存的旧值，主要在数据库发生异常的时候数据库能够回滚 

**事物回滚**

## 为什么不直接写入硬盘

主要是可以根据异步刷入磁盘提升mysql并发能力
# Mysql Buffer Pool

Mysqk Buffer Pool主要用于缓存数据库中的数据页和索引页。当查询的时候会看Buffer Pool是否存储了响应的数据页和索引页，如果有就直接从Pool内存读取替换，而不需要磁盘IO操作，提高了查询性能。

Buffer Pool的优化方案有使用LRU替换策略，这意味着最近访问的会保留在内存中，太久没访问的会被替换出去。

如果更改了Buffer Pool内存太小，可能会导致频繁的IO，如果更改了 Buffer Pool太大，可能导致内存不够影响性能。

## 从一个查询语句来分析Buffer Pool的工作原理

~~~sql
select * from tables1 where id = 1
~~~

这时候Mysql会检查Buffer Pool是否存放了ID为1001的员工信息的数据页，如果该数据已经在Buffer Pool中，Mysql将直接从内存中读取数据，无需IO操作，否则话Mysql就会去磁盘读取数据页（16kb），这回导致IO操作，相比内存IO需要更多的时间。

因为在mysql查询一条的数据的时候，他不是单纯的拿一条数据出来，而是把这条数据的数据页读取出来再拿该数据页的一条。
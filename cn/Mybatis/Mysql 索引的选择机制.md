# Mysql 索引优化

## mysql如何选择使用索引。

~~~sql
select *from user where username = 'a' and password = 'b'
~~~

如果useranme和password都是普通索引，那么他们会把2个索引都查出来，然后在把他们的交集拿出来

如果username是唯一索引，password是普通索引，那么我们的mysql就会优先使用唯一索引



~~~sql
select * from user where a > 2 and b =3
~~~

a和b都是索引,而且还是联合索引

这个时候索引b会失效，因为使用索引a然后在通过回表找里面的b是否等于3

因为联合索引的机制是先从做到有，这个操作在a索引中，他收到的任务是找到所有>2的数据，当大于2的数据都找出来了，那么此时的B是无序的，所以就导致了索引失效，索引不失效的前提的是索引必须保持有序。



~~~sql
select * from user a >= 2 and b = 3
~~~

这时候a和b索引就不会失效，因为a收到的命令是找到>=2的数据，当找到了2以后，然后就去看b，在2内的b索引是有序的，所以这时候b的索引是可以生效的。



~~~sql
ELECT * FROM t_table WHERE a BETWEEN 2 AND 8 AND b = 2
~~~

这个是和我们>= 是一样的，<=一样的不会导致我们的2个索引失效。不同的数据库会有差异，Mysql是这样的



~~~sql
SELECT * FROM t_user WHERE name like 'j%' and age = 22
~~~

name和age他们也是联合索引。首先会找到所有j开头的单词，然后，在挨个判断age是否等于22，也就是age索引会失效。

如果他们都是普通索引，那么mysql会根据索引的复杂度或者索引的大小，唯一值值数量 和 选择性（如j开头的数据比较少） 和索引长度大小（比如值int就是4个字节，varchar(20)就是20个字节)等去选择一个合适的索引去使用

## 什么时候使用索引

字段有一些唯一性，比如username，需要使用where快速定位的。经常用group by和order by的，因为索引都是排序好的就不需要在排序一次了。

## 什么字段不需要索引

where 和group by、 order by用不到的字段，因为索引的价值是快速定位。

当字段有大量重复内容是不需要加索引的。Mysql有一个优化，如果一个字段的内容重复率比较高，那么就会全表扫描。

经常更新的字段不需要建立索引

表数据太少的时候不需要建立索引。

## 优化索引的方案
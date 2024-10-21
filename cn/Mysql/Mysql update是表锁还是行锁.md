# Mysql update是表锁还是行锁

在Innodb引擎下，如果我们有where条件默认是使用行锁，如果没有修改全表默认使用表锁。

在MYISAM情况下默认是使用表锁
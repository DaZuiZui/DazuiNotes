# 单机数据库死亡了会丢数据吗？

## INNODB

Mysql使用事物和写前日志（redo log）来保证数据的一致性， 如果事物已经提交，innodb会写入磁盘，即使mysql死亡了也能通过redo log修复。

未提交的事物就会被回滚，保证事物的一致性。

但是innodb会先写到缓冲区，然后在写到磁盘，如果写入redo log之前死亡了， 可能会丢数据。

Mysql重新启动的时候会自动检查和应用redo log，恢复并写入还没写入磁盘的事物。

### innodb工作流程

1.事物开始

2.在缓冲区修改数据页

3.将所有的修改记录保存到redo log

4.提交事物

5.写入磁盘

## MyISAM

MyISAM存储引擎没有事物和写前日志机制，死亡可能导致数据文件丢失或者丢失，因此MyISAM可靠性不如INNODB

如果死亡了，稚嫩恶搞通过check table和repair table命令来检查和修复。

或者使用二进制日志binlog恢复。


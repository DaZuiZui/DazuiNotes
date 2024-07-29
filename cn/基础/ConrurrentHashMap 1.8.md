# ConrurrentHashMap 1.8

## 如何保证安全

 刚开始会使用CAS操作来进行无锁的并发更新，如果更新失败就会使用synchronized锁在特定的桶和节点上减少锁竞争和性能开销。。


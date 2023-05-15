# 游标遍历清除Redis过期的key

## 为什么要清除Redis过期的Key

    Redis的过期清理是一种懒惰的清理方案，他不会过期后立刻清除，而是在Key被访问的时候进行删除，Redis这么做的目的就是为了提高性能降低资源开销。

    具体来说，一个Key过期的时候，Redis不会立刻从内存中删除。相反我们客户端进行访问的时候Reids会检查当前key是否过期，如果过期就进行删除，这也就是意味着只有被访问的时候才会被删除。而不是真正的过期就立刻删除。

    虽然我们的Reids会自动清除我们过期的Key，但是这个过期清除不是实时的，这个清理过程是在我们访问Key的时候，所以如果有很多Key的过期没有被访问，他们会在内存中存活一段时间，所以一些对过期Key清理要求严格的场景下需要借助其他机制和工具进行清理。

## Redis自身的清除机制

### 读取和写入的时候

    当我们读取和写入的时候我们Reids会检测是否过期，如果过期了就会在读区或写入之前将它立即删除。

### 定期清理

    Reidis会在一定时间间隔执行清理操作，清除过期的Key，这个时间间隔有配置项hz(每秒运行周期性操作的次数)和timeout(扫描时间限制)来决定，默认情况下，每秒运行10次周期性操作。每次周期性操作就是选择一些Key，这些Key可能过期了也可能没有过期，如果过期了就删除。

### 懒惰清理

    每次访问的时候查看这个Key是否过期，如果过期了就把他删除。

## 什么是游标操作或者分页操作

    在Redis游标时一种用于分批处理数据的机制，当需要遍历大量数据或进行大型操作的时候Redis提供了一些分页式的便利方式，使用游标来表示当前遍历的位置，通过多次迭代获取完整的操作。

    游标就是记录当前遍历的位置，一遍下一次遍历的时候从上一次停止的位置进行操作。通过游标可以有效处理大型数据集，避免一次性将所有数据加载到内存中。

    在Rdis可以使用scan进行遍历操作，可以通过传递游标参数来控制遍历的位置，直到游标变成0。

    使用游标的优势就是可以在遍历的过程中处理大量的数据，并且不会对Redis服务器造成过大的负担。游标可以在多个客户端之间进行共享，以支持并发遍历操作。

    需要注意！！！

    游标直表示当前遍历的位置，而不是一个唯一的标识符或键值。在不同的遍历中可能出现相同Key返回多次的情况，我们可以处理遍历结果进行去重进行处理。

## 游标遍历的缺点

### 1.不保证实时性

    游标遍历是一个迭代的操作，通过每次返回的游标来获取下一批数据，这意味着在遍历的过程中，增加和删除都是会重复处理一些Key。

### 2.对Redis服务器资源的占用

    游标遍历需要保持和Redis服务器的链接，而且需要持续发送命令获取下一批数据。这可能占用服务器网络宽带和处理能力，如果遍历过程中可能会持续较长时间，并且占用服务器资源。

### 3.潜在的性能问题

    当数据集很大的时候，使用游标遍历会对我们服务器的性能产生一定的影响。每次遍历迭代都会进行网络通信和执行命令，可能会引入一定的延迟和性能的开销。

### 4.遍历过程中的数据变动。

    Redis是一个并发数据库，遍历过程中可能会有其他客户端对Key进行修改、删除、增加操作，这会影响到我们遍历结果的一致性。

### 5.不适合实时查询

    如果需要实时查询满足特定的条件的Key，游标遍历可能并不是最佳的选择。游标遍历睡一个逐步迭代的过程，需要遍历整个数据集才能找到满足的条件的Key。

## Springboot定时任务+Redis游标处理过期Key

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisConnection;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class ExpiredKeyCleanupTask {
    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    @Autowired
    private RedisConnectionFactory redisConnectionFactory;

    @Scheduled(cron = "0 0 1 * * ?") // 每天凌晨1点执行一次任务
    public void cleanupExpiredKeys() {
        RedisConnection connection = redisConnectionFactory.getConnection();

        try (Cursor<byte[]> cursor = connection.scan(ScanOptions.scanOptions().match("*").count(100).build())) {
            while (cursor.hasNext()) {
                byte[] keyBytes = cursor.next();
                String key = new String(keyBytes);

                if (connection.ttl(keyBytes) <= 0) {
                    redisTemplate.delete(key);
                }
            }
        }
    }
}

```

1. 在上述代码中，使用了 `@Autowired` 注解将 `RedisTemplate` 和 `RedisConnectionFactory` 注入到定时任务类中。
2. 使用 `@Scheduled` 注解标记 `cleanupExpiredKeys` 方法为定时任务，并指定执行时间表达式，例如 `cron = "0 0 1 * * ?"` 表示每天凌晨1点执行任务。
3. 在 `cleanupExpiredKeys` 方法中，执行游标遍历逻辑，使用 `scan` 方法遍历 Redis 键空间，并根据 `ttl` 方法判断键是否过期，如果过期则使用 `delete` 方法删除键。
4. 可根据需要调整时间表达式和其他参数，确保定时任务按预期执行。

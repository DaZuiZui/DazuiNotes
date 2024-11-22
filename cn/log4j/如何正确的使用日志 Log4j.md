# 	如何正确的使用日志 Log4j

参考https://leetcode.cn/circle/discuss/feCy4y/

**TRACE：**最细粒度的信息，通常只在开发过程中使用，用于跟踪程序的执行路径。
**DEBUG：**调试信息，记录程序运行时的内部状态和变量值。
**INFO：**一般信息，记录系统的关键运行状态和业务流程。
**WARN：**警告信息，表示可能存在潜在问题，但系统仍可继续运行。
**ERROR：**错误信息，表示出现了影响系统功能的问题，需要及时处理。
**FATAL：**致命错误，表示系统可能无法继续运行，需要立即关注。

~~~java
// 推荐
logger.debug("用户ID：{} 登录成功。", userId);
~~~

~~~java
try {
    // 业务逻辑
} catch (Exception e) {
		logger.error("处理订单ID：{} 时发生异常：", orderId, e);
}
~~~

~~~java
if (index % 1000 == 0) {
    logger.info("已处理 {} 条记录", index);
}
~~~

## 控制日志输出量

设置好日志过滤，防止刷屏

~~~java
<!-- Logback 示例 -->
<appender name="LIMITED" class="ch.qos.logback.classic.AsyncAppender">
  	<!-- 只允许 INFO 级别及以上的日志通过 -->
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
        <level>INFO</level>
    </filter>
    <!-- 配置其他属性 -->
</appender>

~~~

## 控制入口出口

很多开发者（尤其是线上经验不丰富的开发者）并没有养成记录日志的习惯，觉得记录日志不重要，等到出了问题无法排查的时候才追悔莫及。

一般情况下，需要在系统的关键流程和重要业务节点记录日志，比如用户登录、订单处理、支付等都是关键业务，建议多记录日志。

**对于重要的方法，建议在入口和出口记录重要的参数和返回值，便于快速还原现场、复现问题。**

对于调用链较长的操作，确保在每个环节都有日志，以便追踪到问题所在的环节。

比如可以利用 AOP 切面编程在每个业务方法执行前输出执行信息：

~~~java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.example.service..*(..))")
    public void logBeforeMethod(JoinPoint joinPoint) {
        Logger logger = LoggerFactory.getLogger(joinPoint.getTarget().getClass());
        logger.info("方法 {} 开始执行", joinPoint.getSignature().getName());
    }
}
~~~

注意不要在日志中记录了敏感信息，比如用户密码。万一你的日志不小心泄露出去，就相当于泄露了大量用户的信息。

## 日志的管理

随着日志文件的持续增长，会导致磁盘空间耗尽，影响系统正常运行，所以我们需要一些策略来对日志进行管理。

首先是设置日志的滚动策略，可以根据文件大小或日期，自动对日志文件进行切分。比如按文件大小滚动：

~~~java
<!-- 按大小滚动 -->
<rollingPolicy class="ch.qos.logback.core.rolling.SizeBasedRollingPolicy">
    <maxFileSize>10MB</maxFileSize>
</rollingPolicy>
~~~

如果日志文件大小达到 10MB，Logback 会将当前日志文件重命名为 app.log.1 或其他命名模式（具体由文件名模式决定），然后创建新的 app.log 文件继续写入日志。

还有按照时间日期滚动：

~~~java
<!-- 按时间滚动 -->
<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
    <fileNamePattern>logs/app-%d{yyyy-MM-dd}.log</fileNamePattern>
</rollingPolicy>
~~~

上述配置表示每天创建一个新的日志文件，%d{yyyy-MM-dd} 表示按照日期命名日志文件，例如 app-2024-11-21.log。

还可以通过 maxHistory 属性，限制保留的历史日志文件数量或天数：

~~~java
<maxHistory>30</maxHistory>
~~~

这样一来，我们就可以按照天数查看指定的日志，单个日志文件也不会很大，提高了日志检索效率。

对于用户较多的企业级项目，日志的增长是飞快的，因此建议开启日志压缩功能，节省磁盘空间。

 ~~~java
 <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
     <fileNamePattern>logs/app-%d{yyyy-MM-dd}.log.gz</fileNamePattern>
 </rollingPolicy>
 ~~~

## 统一日志格式

建议每个项目都要明确约定和配置一套日志输出规范，确保日志中包含时间戳、日志级别、线程、类名、方法名、消息等关键信息。

~~~java
<!-- 控制台日志输出 -->
<appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
        <!-- 日志格式 -->
        <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
</appender>
~~~

## 异步日志

~~~java
<!-- 异步 Appender -->
<appender name="ASYNC" class="ch.qos.logback.classic.AsyncAppender">
    <queueSize>500</queueSize> <!-- 队列大小 -->
    <discardingThreshold>0</discardingThreshold> <!-- 丢弃阈值，0 表示不丢弃 -->
    <neverBlock>true</neverBlock> <!-- 队列满时是否阻塞主线程，true 表示不阻塞 -->
    <appender-ref ref="CONSOLE" /> <!-- 生效的日志目标 -->
    <appender-ref ref="FILE" />
</appender>

~~~

## 日志分析

在比较成熟的公司中，我们可能会使用更专业的日志管理和分析系统，比如 ELK（Elasticsearch、Logstash、Kibana）。不仅不用每次都登录到服务器上查看日志文件，还可以更灵活地搜索日志。

但是搭建和运维 ELK 的成本还是比较大的，对于小团队，我的建议是不要急着搞这一套。


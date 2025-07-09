# SkyWalking分布式链路追踪

其实在SAAS开发他主要就是做了链路追踪和关键业务中做埋点的。

其实主要就几个部分组成吧

skyWalking的基本链路追踪原理。

Trace、Span、Segment概念

自动埋点(Agent方法)

手动埋点

数据上报与链路拼接原理

## 1.链路追踪点基本原理

SkyWalking的链路追踪主要依赖 分布式调用链，通过服务调用的每一环节记录请求信息，形成完整的调用链接。

他的工作原理主要就是

​	1.每个请求进入系统的时候生成一个TraceID（表达一个完整的调用链路的唯一表示）

​	2.每个服务（或者进程）中由skywalking agent拦截请求，建立span（表示一次调用调用或者方法执行）	

​    3.通过ContextCarrier在RPC、HTTP、MQ等边界传递上下文，爆炸TraceID在不同服务间传递。

​	4.所有采集的数据（Segment）最终发送到OAP Server聚合，形成可视化的链路，方便运维和开发人员查看。

## 2.Trace、Span、Segment概念

### 2.1 Trance（调用链）

表达一个完成的调用链。比如用户/order/create。然后记录整个调用链。

一个Trace包含多个Span

### 2.2 Segment 服务调用段

表示某个服务实列内部产生的调用片段。

Span类型：

​	服务入口
​	服务出口（比如HTTP调用下游）

​	Localspan： 服务内部逻辑

## 3. 自动埋点（Agent方法）

SkyWalking虽然对常用的框架提供了自动探针，无需更改代码就可以采集链路。

1.想使用就是在JVM启动参数添加-javaagent:/path/to/skywalking-agent.jar

2.Agent会使用字节码增强（ByteDuddy）或者字节码注入ASM 拦截常用方法

3. 自动创建EntrySpan和ExitSpan 自动上报OAP

## 4. 手动埋点

有些业务不会被自动探针捕捉，只需要一个注解+  ActiveSpan.tag就可以手动埋点了

~~~java
import org.apache.skywalking.apm.toolkit.trace.Trace;
import org.apache.skywalking.apm.toolkit.trace.ActiveSpan;

@Service
public class OrderService {

    @Trace(operationName = "createOrder") // 声明一个自定义 Span
    public void createOrder(Order order) {
        ActiveSpan.tag("userId", order.getUserId()); // 打标签，方便查询
        ActiveSpan.tag("orderId", order.getId());
        
        // 关键业务逻辑
        checkInventory(order);
        saveOrder(order);
        notifyPayment(order);
    }
}
~~~

## 5. 数据上报和链路拼接

Agent收集数据：每个服务的探针会把SPan存在内存缓冲区。

跨服务传递TraceId/

​	HTTP：通过Header（sw8）传递上下文

​	RPC：通过拦截器注入上下文

OAP Server 汇聚链路

​	接受Agent上报的Segment

​	根据TraceId + segmentId拼接完整的链路。

​	持久化保存

然后SkyWalking 展示调用链。


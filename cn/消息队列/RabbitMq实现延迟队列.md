# RabbitMq实现延迟队列

Rabbitmq要想实现消息延迟队列需要rabiitmq插件（Delayed Message Exchange 插件）在java中

~~~xml
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>${rabbitmq-client-version}</version>
</dependency>
~~~

## delayed message实现延迟队列的秘密

**交换机与队列的绑定；** 首先他可以让我们创建一个特殊类型的交换机Delayed message excahnge。然后与这个交换机与我的队列进行绑定。

**消息的存储：** 当我们发送一个消息到我们交换机的时候不会立刻将消息给我们的队列的，而是会将消息存储到我们的内部队列中。

**延迟消息的管理：** 插件会根据我们消息的TTL(Time to live 消息的生存时间)来管理延迟时间。这个TTL就是我们发送消息设置的延迟时间。

**定时器：** 他会定期检查存储的消息是否超过了设定的延迟时间，一旦超过了就会从内部队列取出来放到我们绑定的工作队列。

**消息路由：** 在消息投递到绑定队列之前，dalayed交换机会根据消息的路由key和绑定的队列进行匹配，已确定消息最后投递到哪个队列中。

## 使用代码

**生产者：**

~~~java
public void sendMessageWith3HourDelay(String message) {
    // 将3小时转换为毫秒
    long delayInMillis = 3 * 3600 * 1000;
    
    rabbitTemplate.convertAndSend("delayed.exchange", "delayed.queue", message, messagePostProcessor -> {
        messagePostProcessor.getMessageProperties().setDelay((int) delayInMillis);
        return messagePostProcessor;
    });
}

~~~

**消费者:**

~~~java
// 消费者应用程序
@EnableBinding(Sink.class)
public class MessageConsumer {

    @StreamListener(Sink.INPUT)
    public void handleMessage(String message) {
        System.out.println("Received message: " + message);
    }
}
~~~


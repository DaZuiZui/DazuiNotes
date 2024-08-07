# RabbitMq如何保证消息的可靠性和稳定性

rabbitMq不会百分之百让我们的消息安全被消费，但是rabbitMq提供了一些机制来保证我们的消息可以被安全的消费。

## 消息确认

消息者在成功处理消息后可以发送确认（ACK）给rabbitMq，通知消息已经被成功处理。如果RabbitMq没有收到确认，它可以重新发消息给另外一个消费者。这样可以防止消息丢失。

## 持久化

生产者可以将消息标记为持久的， 这样哪怕我们重启了也不会丢消息。队列可也可以标记为持久的。

## 发布确认

他就是生产者确认消息已经被交换机接受并且已经被消费者接受。

## 死信

如果消息因为某些原因无法被处理（例如，消息被拒绝并且不在重新投递），他可以被发送到死信队列进行进一步处理或者分析。

## 集群会高可用性

rabbitmq支持集群和镜像队列，来提高可靠性和可用性，在集群中可以实现消息和队列在多个节点间复制，防止单点故障消息丢失。

### 镜像队列

镜像队列就是主队列的副本。他和主从的却别就是主从式异步同步，他说同步复制
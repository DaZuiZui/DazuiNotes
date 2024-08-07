# TCP 建立链接

## 建立链接 - 三次握手

**1.客户端发送SYN报文段（同步请求)给客户端，包含了客户端的初始序列号（INS）**

**2.服务器接收了SYN后，发送ACK报文段（确认），确认了客户端的序列号，并发送自己的SYN报文段，建立链接。**

**3.客户端收到了服务器的SYN和ACK，发送ACK报文确认，建立完成**

### ISN

​	这个序列号就是用来确认字节流中的数据位置的，比如第一次初始序列号是10086，然后数据段长度是2000，第一次传了1000，然后序列号变成11086，然后再传变成了12086，然后他发现我们接受的数据大小和content-length一样大了，就判定发送结束了。

​	这么做的原因就是防止攻击者预判和干扰通信。TCP序列号预测攻击，TCP重放攻击（一个段被成功处理了又来了一次），会话劫持（猜到我们的序列号了），保证数据的完整性。

### 精简版

**SYN**：

- 客户端发送SYN（同步序列号）包到服务器，请求建立连接。

**SYN-ACK**：

- 服务器收到SYN包后，回复一个SYN-ACK包，表示同意连接，并为客户端分配初始序列号。

**ACK**：

- 客户端收到SYN-ACK包后，发送一个ACK包确认，连接正式建立，双方进入ESTABLISHED状态。

## 关闭链接-四次挥手

### 第一次挥手(FIN)

客户端发现我当天发送的字节长度和请求头的content-length一样长了，知道自己发完了，然后发送一个FIN（结束）包，表示已经完成传输。我不在发送数据了，但是我依然接受数据。

这个FIn包含了一个序列号(假设为'u')

然后客户端进入close_wait_1状态

### 第二次挥手(ACK)

服务器收到了FIN包，然后发送一个ACK包。（u+1）

然后服务器进入了Close_wait状态。我不在发送数据，但是我依然接受数据。

## 第三次握手(FIN)

服务器也完成数据传输后，然后发送一个FIN包，我不在发送任何数据。

### 第四次挥手 ACK

客户端收到FIN包，发送一个ACK包， 然后进入time_wait状态，等待一段时间后（一般2分钟），连接正式关闭。



## 如果没有第三次握手

如果没有第三次握手然后我们客户端也没发生ack包。

## 链接没打开

服务器处于半打开状态，它认为连接已经建立，但客户端实际上认为连接还没建立。

服务器的资源被占用，因为服务器已经为该连接分配了资源，但是无法通信。

服务器会为每个半打开的连接保留资源，这些资源可能在长时间无法释放， 导致资源浪费。

## 数据传输不可靠

服务器和客户端之间的状体不同步。客户端未确认连接的建立，不会发送数据，服务器等待客户端的确认，无法接受数据。

可能会导致数据丢或者传输延迟，因为双方未能确认连接状态。

## 安全风险

攻击点可能利用这一点进行SYn Flood攻击，发送大量的SYN请求但不回应SYN-ACK，导致服务器资源耗尽，无法处理正常的请求。
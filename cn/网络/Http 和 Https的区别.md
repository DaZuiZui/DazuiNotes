# Http 和 Https的区别

## Http

Http是一种不安全的协议，数据在传输过程都是明文传输的，

1.当客户端需要与服务器建立连接的时，它会发送一个http请求，服务器接受请求后会返回一个http响应，这就是达成了初次握手。

2.Http默认是短连接的，每个http响应周期后就会被立刻关闭。

## Https

### 三次挥手

**1.client hello：**客户端发送支持的TLS/SSL、加密套件、还有客户端随机数

​		这一步是明文发生，攻击者可以看到客户端的随机数。

**2.ServerHello：**   服务器选择TLS/SSL版本和加密套件，发送服务器随机数和服务器额度数字证书

​		这一步也是明文的，攻击者也可以看到服务器随机数和服务器公钥。

**3.客户端验证服务器身份：** 客户端验证服务器身份 by 电子证书

​		黑客无法伪造。

**4. 预主密钥** 客户端生成预主密钥，然后用服务器的公钥加密发送给服务器

​		这内容只有服务器私钥可以解密。

**5.服务器解析预主密钥** 服务器使用自己的私钥解密客户端发送的预主密钥。获取到明文的预主密钥。

**6.双方通过 双方的随机数和预主密钥（解密后的）**		生成对称密钥

**7. 双方交换加密的finished消息，确认握手过程的完整性和正确性。**
    “Finished”消息用于验证握手是否完整、没有被篡改。
     如果对方能正确解密并验证消息，握手完成。
​	  消息是用的对称密钥加密的

​		

  

   客户端和服务器端互相通知握手完成，安全通信开始。

### 四次挥手

​	客户端向服务器发送一个关闭通知表示我不在向你发起数据，但是仍然愿意接受数据

​	服务器收到客户端的关闭通知，向客户端确定关闭通知，此时，服务器不再发送数据，但是仍然愿意接受数据。

 	服务器想客户端发送关闭通知，表示服务器不再发送数据。

​	客户端向服务器确定关闭通知，此时，双方正式关闭。

# Springboot websocket前端无法访问到，Websocket因AOP代理 前端无法请求到

## 问题出现

 在我后端springboot启动后，前端无法请求websocket请求连接到我们websocket服务器。

## 想要的效果

 在我后端springboot启动后，前端可以请求到我们websocket服务器，并且进行交互。

## 问题排查

### 出现的问题A

**出现问题的代码：**

```java
package com.gsxy.core.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config){
        config.enableSimpleBroker("/topic");
        config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry){
        registry.addEndpoint("/ws").withSockJS();
    }

}
```

问题出在没有告诉spring遇见wensocket协议该如何处理。



**改正后的代码：**

```java
package com.gsxy.core.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.server.standard.ServerEndpointExporter;

@Configuration
@EnableWebSocket
public class WebSocketConfig  {

    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }

}
```



### 出现的的问题B

问题出在Websocket在controller包下，在我们aop SystemAopImpl，对我们的contorller包下进行了所有代码增强。

```java
/**
 * @author zhuxinyu 2023-10-23
 */
@Component
@Aspect
public class SystemAopImpl implements SystemAop {
    /**
     * @author zhuxinyu 2023-10-23
     * 清理ThreadLocal 防止内存泄漏
     * @param joinpoint
     * @throws Exception
     */
    @Override
    @After("execution(* com.gsxy.core.controller.*.*(..))")
    public void removeAllThreadLocal(JoinPoint joinpoint) throws Exception {
        ThreadLocalUtil.mapThreadLocalOfJWT.remove();
        ThreadLocalUtil.mapThreadLocal.remove();
        ThreadLocalUtil.DataOfThreadLocal.remove();

    }

}
```

​	因为在Spring AOP中使用的是IOC 和 AOP动态代理创建对象，在WebSocket中，如果代理类被代理了，可能会出现问题因为WebSocket容器会查找类上的注解，但是无法找到代理类上，因为@ServerEndpoint来自Java标准注解，并不是AOP，如果我们使用的是cglib动态代理技术，执行的是目标类的子类，这个字类包含我们的拦截逻辑和目标方法的引用，所以无法读取到websocket的注解，导致无法访问websocket，因为我们的websocket没有实现接口，默认实现的cglib动态代理技术，所以触犯了这个问题。jdk动态代理，他执行的是代理对象，这个代理类是包含了我们的环绕逻辑和我们目标类的代理对象，所以不会导致注解失效。

## 总结

使用websocket不能被cglib所代理。

必须告诉spring遇见websocket如何解决
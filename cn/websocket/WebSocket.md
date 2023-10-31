# WebSocket

## 后端代码

```java
package com.dazuizui.business.websocket;

import com.dazuizui.business.service.onlineJudge.CompetitionInfoService;
import com.dazuizui.business.service.user.UserService;
import com.dazuizui.business.util.SpringContextUtil;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.CrossOrigin;

import javax.websocket.OnClose;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;
import java.io.IOException;

@CrossOrigin
@ServerEndpoint("/api/zuioj/{contest_id}/{page}/{size}")
@Component
public class RandomNumberController {

    @Autowired
    private CompetitionInfoService competitionInfoService = SpringContextUtil.getBean(CompetitionInfoService.class);

    @OnOpen
    public void onOpen(Session session, @PathParam("contest_id") Long contestId , @PathParam("page")Integer page , @PathParam("size")Integer size) throws IOException, InterruptedException {
        System.out.println("连接成功" + contestId+" and "+page+" "+size);
        String returnJSON = competitionInfoService.viewRanking(contestId, page, size);
        while (true){
            Thread.sleep(1000);
            session.getBasicRemote().sendText(returnJSON);
        }

    }

    @OnClose
    public void onClose(Session session) {
        System.out.println("closed");
    }

    @OnMessage
    public void onMessage(String message, Session session) throws IOException {
        session.getBasicRemote().sendText("asd");
    }
}
```

## 前端代码

~~~js

        setupWebSocket() {
            const contestId = 80; // 用于示例的contest_id

            this.socket = new WebSocket(`ws://127.0.0.1:8001/api/zuioj/${contestId}/1/25`);

            this.socket.onopen = () => {
                this.socketStatus = '已连接';
            };

            this.socket.onmessage = (event) => {
                this.receivedMessage = event.data;
                console.log(event);
            };

            this.socket.onclose = () => {
                this.socketStatus = '已关闭';
            };
        },
~~~

## WebSocket使用autowired注入后为null

因为在默认情况下，@Bean是singleton作用域的， 在singleton作用域的生命周期开始是在应用程序开始的时候生命周期开始，但是websocket控制器默认作用域是prototype，这意味着每次这意味着每次创建WebSocket对象时都会重新注入Service这个Bean，但是Service的Bean是单列的只有在ioc启动的时候才会被创建所以导致了使用autowired注入后为null。

### 解决方案A

根据Spring上下文进行依赖注入。

```java
private CompetitionInfoService competitionInfoService = SpringContextUtil.getBean(CompetitionInfoService.class);
```

**工具类代码**

```java
package com.dazuizui.business.util;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

@Component
public class SpringContextUtil implements ApplicationContextAware {

    private static ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        SpringContextUtil.applicationContext = applicationContext;
    }

    public static <T> T getBean(Class<T> beanClass) {
        return applicationContext.getBean(beanClass);
    }

    public static <T> T getBean(String beanName, Class<T> beanClass) {
        return applicationContext.getBean(beanName, beanClass);
    }
}
```

### 方案B

如果被其他类也在使用可以把业务类的作用域也设置为prototype

### 方案C

如果该业务类是为websocket服务可以设置为websocket作用域
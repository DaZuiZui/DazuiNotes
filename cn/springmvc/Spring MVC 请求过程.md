# Spring MVC 请求过程

## 1.客户段请求

客户端发送一个HTTP请求，这个请求包括URL、HTTP方法（Get Post等），请求头（Accept、content-type等），和请求体（Post请求下存在）

## 2.DispatcherServlet接受请求

在Spring MVC中，所有请求最先到达的是DispatcherServlet。DispatcherServlet是一个Servlet，他充当一个前端控制器的角色（前端控制器是一种设计模式）。负责统一请求的分发和处理流程。

## 3.HandlerMapping

DispacherServlet根据请求信息(URL)通过HandlerMapping找到对应的Handler。然后HandlerMapping将请求映射到对应的Controller或者处理器方法上。

## 4.HandlerAdapter

一旦找到对应的handler了，我们的DispacherServlet就会将请求交给HandlerAdapter调用实际Controller中的方法。

## 5.处理器处理业务

处理器进行业务处理，如果我们不走视图解析器那么到这里就结束了，把json返回给dispacherservlet然后返回给客户端

## 6.ModelAndView

如果走了视图解析器会返回一个ModelAndView对象。Model是数据处理结果的载体。View是视图的名字。

## 7.viewResolver

dispatcherServlet使用ViewResolver将逻辑视图名字解析为具体的视图名字。视图对象负责渲染仕图页面，将数据模型渲染填充进去。

## 8.视图渲染

最后渲染出来的视图页面给客户端，这个页面可能是HTML也可能是JSP。

## 9.响应给客户端。
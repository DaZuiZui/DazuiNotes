# MVC工作原理

## 有视图的情况

1.客户端（浏览器）发起请求，DispatcherServlet拦截请求。

2.DispatcherServlet根据请求信息调用HandlerMapping。HandlerMapping根据uri去匹配查询能处理的Handler（也就是我们所说的Controller），并且将会我们涉及到的拦截器和Handler一起封装。

3.DIspatcherServlet调用HandlerAdapter适配器执行Handler。

4.Hander完成用户请求的处理后回返回一个ModelAndView对象给DispatcherServlet。

​		Model是返回的数据对象

​		View是视图对象，如JSP

5.ViewResolver回根据逻辑View查询实际的View.

6. DispaterServlet把返回的Model传给View视图渲染
7. 把View返回给请求者。

## 没有使用视图的情况

​	1.客户端发送请求到DispatcherServlet。

​	2.DispatcherServlet根据HandlerMapping找到对应的Controller

​	3.如果Controller被标记不走视图解析器那就该返回值直接作为响应内容。

​	4.Spring MVC通过消息转换器将Java对象转换为JSON格式的响应数据。

​	5.将JSON数据作为HTTP响应返回给客户端。

## 核心组件

### DispacherServlet 调度器

​	核心的中央处理器，接受所有的HTTPS请求，并将请求分发给响应的处理器进行处理。

### HandlerMaping

根据uri去匹配能处理的Handler，并将请求涉及到的拦截器和Handler一起封装起来。

### Handler处理器

负责处理具体的请求。他可以是一个Controller类，也可以是一个带有注解的方法。

### HeadlerAdapter 处理器适配器

​	用于适配执行Handler。HanlderAdapter负责解释请求参数、数据绑定、验证等操作，以及调用Handler的相应方法进行业务处理。

### ViewResolver 视图解析器

​	根据视图名字解析出对应的View对象，他负责将模型数据和视图结合起来生成最终响应的HTML。

### HandlerExceptionResolver 异常处理器

​	处理请求处理过程中产生的异常，它负责将异常转换为合适的错误响应，或者执行其他自定义异常处理逻辑。

### MessageConverter 消息转换器

​	负责处理请求和响应的消息转换，它可以将Java对象转换为JSON、XML等格式，以便在请求和响应之间进行数据的传输和转换。

​	它主要用于消息请求体上的数据转换为处理器方法所需要的参数类型，如Post、Put。

​	如果Get请求一般不会走MessageConverter的，他是通过Spring MVC的参数自动映射来转换参数的，除非使用特定格式的字符串进行传递

# Spring 的异常处理机制

​	在Spring中，异常处理是一个非常重要的方面，用于捕获和处理应用程序中可能出现的异常情况。Spring提供了多种方式来处理异常。

​	使用Spring的异常处理机制主要有以下优点：

​			**统一的异常处理：**通过全局异常处理器，可以实现一致的异常处理逻辑，而不需要在每个控制器或方法中添加异常代码。

​			**错误信息的友好展示：**可以将错误信息转换为友好的错误页面或者JSON响应。

​			**日志处理：**Spring的异常处理通常和日志记录继承，可以记录程序中的异常，以便于后期分析和排查问题。

​			**提高可维护性：**通常将异常处理逻辑集中在一个地方，可以提高代码的可维护性、降低代码的重复性。

​		

## 使用try-catch

​	就像在Java程序中一样来捕获异常，但是Spring中，通过不会直接使用try-catch，而是依赖于Spring的异常处理机制。

## Spring的全局异常处理

​	如果使用Spring全局异常处理是需要实现接口HandlerExceptionResolver，或者使用@ControllerAdvice注解来实现。

~~~java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public ModelAndView handleException(Exception ex) {
        // 处理异常并返回一个ModelAndView
        ModelAndView modelAndView = new ModelAndView("error");
        modelAndView.addObject("errorMessage", ex.getMe	ssage());
        return modelAndView;
    }
}
~~~

## Springboot的默认异常处理。

​	如果使用Springboot，他内置了一个默认的全局异常处理机制， Springboot会处理这些未处理的异常，并且将它记录到日志中，我们也可以自定义错误页面，也可以使用ErrorController来处理这些异常。

## 使用AOP异常处理

​	使用Aop的异常环绕来处理异常处理。
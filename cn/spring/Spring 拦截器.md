# Spring 拦截器

Spring拦截器是用于Controller之前之后的进行拦截和处理机制类似我们Servlet中的过滤器，可以做一些鉴权呀，日志等操作。

## HandlerInterceptor

`preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)`：在请求处理之前调用，返回`true`继续处理请求，返回`false`中止请求处理。

`postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)`：在请求处理之后，生成视图之前调用。

`afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)`：在整个请求结束之后调用，通常用于资源清理。

**实现自定义拦截器**：

- 实现`HandlerInterceptor`接口或继承`HandlerInterceptorAdapter`类（该类已被弃用，建议直接实现`HandlerInterceptor`接口）。

## 实现步骤

定义拦截器

~~~java
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

public class MyInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 在请求处理之前执行的逻辑
        System.out.println("Pre Handle method is Calling");
        return true; // 返回true继续处理请求，返回false中止请求
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        // 在请求处理之后，生成视图之前执行的逻辑
        System.out.println("Post Handle method is Calling");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        // 在整个请求完成之后执行的逻辑
        System.out.println("Request and Response is completed");
    }
}
~~~

**注册拦截器**

~~~java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyInterceptor())
                .addPathPatterns("/**") // 拦截所有请求
                .excludePathPatterns("/login", "/register"); // 排除特定请求
    }
}

~~~


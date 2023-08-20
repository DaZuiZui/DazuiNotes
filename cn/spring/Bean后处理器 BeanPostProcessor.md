# Bean后处理器 BeanPostProcessor

​	Bean后处理器主要用于Spring容器初始化前后执行自定义操作和

​		**修改Bean属性：**可以检查和修改Bean的属性值，这样可以动态地更改Bean的配置

​		**AOP代理：**也可以使用Bean后处理器为Bean创建AOP代理。

​		**资源注入：** 可以注入资源、依赖项或其他Bean，以满足Bean的运行时的需求

​	**Spring后处理器有自己的生命周期，与Spring容器的生命周期相关。**

​		**1.实力化**：Bean后属处理首先在Spring 容器启动的时候实列化。

​		**2.调用postProcessBeforeInitialazation方法**在初始化前

​		3.初始化

​		4.**调用postProcessAfterinitialization方法** 在初始化后调用调用postProcessAfterInitialization方法

​		5.使用阶段

​		6销毁： 如果配置了销毁方法那么就进行销毁	

他和Bean的生命周期十分像，不过Bean后处理器是由Spring IOC容器启动的时候实力化的，而Bean是IOC进行的。
# @ModelAttribute

当我们使用Params传值的时候，我们后端是已对象进行接受

```java
public R queryByPage(@ModelAttribute Patrol patrol, @RequestParam("size") Integer size,@RequestParam("page") Integer page)
```

加了这个对象Spring就会尝试从这个模型中查看与前端传递的Parmas匹配的值进行数据绑定。

## 如果我们对象有个值不能为null怎么半

那就在对应的属性写@NotNull

~~~java
@NotNull
prviate Integer id;
~~~




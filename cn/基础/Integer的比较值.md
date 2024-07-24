# Integer的比较值

## int和Integer

~~~java
int a = 200;
Integer b = 200;
System.out.println(a == b);
~~~

这时候会对我们b进行拆箱转换为int类型，拆箱后比较的是连个基本的int类型。也就是200==200返回的true。



## Integer和Integer的情况

如果是-128 到127区间那么比较是true，因为integer会缓存。

~~~java
Integer a = 123;
Integer b = 123;
System.out.println(a == b); // 这是在比较对象引用。
~~~

如果不是这个区间，那么返回的就是false，因为他们没有缓存是2个不同的对象


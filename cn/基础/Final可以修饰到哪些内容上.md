# Final可以修饰到哪些内容上

final关键字可以增加代码的安全性和和可读性，尤其在确保某些内容不可以被修改不可以被继承。

## 方法参数

保证了不能再方法内部修改

~~~java
void process(final int num) {
    // 不能在方法内部修改num的值
}
~~~

## 局部变量

~~~java
void calculate() {
    final int x = 10;
    // 不能修改x的值
}
~~~

不能修改x值在这个方法内

## 类

~~~java
final class FinalClass {
    // 内容
}

// 无法继承FinalClass类，因为它被final修饰
class ChildClass extends FinalClass {
    // 内容
}
~~~

保证了类不可以被继承

## 方法

~~~java
class Parent {
    final void display() {
        System.out.println("Parent's display method");
    }
}

class Child extends Parent {
    // 无法重写display方法，因为它被final修饰
}
~~~

保证方法不可以被重写

## 变量

~~~java
final int MAX_VALUE = 100;
~~~

保证赋值了就不能被修改
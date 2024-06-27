# 深入探讨java多态

​	多态(Polymorphism)是面向对象一个特别重要的概念，它可以让不同的对象已相同的方式响应方法调用，提高了代码的灵活性和可维护性。多态主要有2种形式：编译时多态性（静态多态性）和运行时多态性（动态多态性），其中后者是最常见的。

## 编译时多态

~~~java
public void print(int num) {
    System.out.println("Printing an integer: " + num);
}

public void print(String text) {
    System.out.println("Printing a string: " + text);
}

~~~

编译时多态是指在编译阶段确定方法的调用，它与方法重载相关。

重载就是方法名相同，但是参数不同。编译器会根据方法的参数类型和数量来选择调用哪个方法。

## 运行时多态（动态多态）

运行时多态是指程序运行时根据对象的实际类型来确定方法的调用，他与方法重写相关。

重写是指子类重新定义了父类中一存在的方法，子类的方法方法名字和参数列表和返回类型都和父类一样。

运行时多态就是调用一个方法程序会根据对象的实际类型来决定调用哪个版本的方法。

~~~java
class Animal {
    public void makeSound() {
        System.out.println("Animal makes a sound");
    }
}

class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Dog barks");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal myAnimal = new Dog();
        myAnimal.makeSound(); // 这里会调用Dog类中的makeSound方法
    }
}

~~~

## 如果动态多态方法是静态会发生什么

~~~java
class A {
    static void display() {
        System.out.println("Static display method in class A");
    }
}

class B extends A {
    static void display() {
        System.out.println("Static display method in class B");
    }

    static void uniqueMethod() {
        System.out.println("Unique static method in class B");
    }
}

public class Main {
    public static void main(String[] args) {
        A a = new B(); // 多态引用
        a.display();   // 调用的是 A 的静态方法，因为静态方法在编译时绑定
        
        // 调用 B 的静态方法
        B.display();   // 直接使用类名调用
        
        // 调用 B 的特有静态方法
        B.uniqueMethod();
    }
}
~~~

调用的是A的静态方法，因为display方法属于类，他在编译的时候就确定了，是通过引用变量的类型决定的，而不是引用实际对象类型。

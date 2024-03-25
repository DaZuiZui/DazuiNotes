# 为什么重写equals必须重写hashcode?

已HashMap和HashSet为例子吧，如果我们重写equals方法没有重写Hashcode回导致Hash 表无法正确的存储对象，因为他俩会根据Hashcode来确定存储位置，如果我们没重写可能会根据Object的类而实现，及时通过我们的equals判断是相同的，但是他们的hashcode也可能不同，导致我们Hash表无法正确存储，可能会出现一些重复的k-v，也会无法正确的找到对象。

~~~java
import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;
import java.util.Set;

//出现重复的k-v案例
class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Person person = (Person) obj;
        return age == person.age && name.equals(person.name);
    }

    // We don't override hashCode() method
}

public class Main {
    public static void main(String[] args) {
        Person p1 = new Person("Alice", 30);
        Person p2 = new Person("Alice", 30);

        Map<Person, String> map = new HashMap<>();
        map.put(p1, "Value1");
        map.put(p2, "Value2");

        System.out.println("Size of map: " + map.size()); // Output: Size of map: 2

        Set<Person> set = new HashSet<>();
        set.add(p1);
        set.add(p2);

        System.out.println("Size of set: " + set.size()); // Output: Size of set: 2
    }
}
~~~



如果我们要进行重写要保证2个对象通过equals方法判断相等，他们的hashCode也要返回相同的值。这样才能保证对象相等和哈希值的一致性，确保使用集合不发生意外。
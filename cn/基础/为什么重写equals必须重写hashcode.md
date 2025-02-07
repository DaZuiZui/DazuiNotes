equals就是判断两个对象是否相等，**Hashcode就是返回对象的哈希码，用于基于Hash的数据结构定位对象。**

他们的约定就是**如果equals方法相同那么他们hashcode必须相同**。如果hashcode相同他们equals未必相等，他们会被放到同一个桶里，然后再用equals方法比较。

也就是说重写equals方法必须重写hashcode。比如hashmap来说，如果equlas相同，他们的hashcode也必须相同，如果重写了equals不重写hashcode，可能会出现问题。

# 为什么重写equals必须重写hashcode?

已HashMap和HashSet为例子吧，如果我们重写equals方法没有重写Hashcode会导致Hash 表无法正确的存储对象，因为他俩会根据Hashcode来确定存储位置，如果我们没重写可能会根据Object的类而实现，即使通过我们的equals判断是相同的，但是他们的hashcode也可能不同，导致我们Hash表无法正确存储，可能会出现一些重复的k-v，也会无法正确的找到对象。

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

简单而言，比如两个人都叫张三，他们equal肯定为ture，但是他们并不是同一个人，hashcode更像他们的IC来区别

## equals和hashcode的检查顺序

是先判断hashcode然后在判断equals更加精确的判断。

## 场景题

~~~java
User user1 = new User("user1",18,1);
User user2 = new User("user1",18,1);
User user3 = new User("user3",19,1);
怎么满足map.get(1) == map.get(user2) !=  map.get(user3)
~~~

`map.get(1)` 的结果需要等于 `map.get(user2)`，这说明 `1` 和 `user2` 的键需要被视为相等。

`map.get(user2)` 和 `map.get(user3)` 不相等，这说明 `user2` 和 `user3` 的键不能被视为相等。

~~~java
import java.util.HashMap;
import java.util.Map;
import java.util.Objects;

public class Main {
    public static void main(String[] args) {
        // 自定义 User 类
        class User {
            String name;
            int age;
            int id;

            public User(String name, int age, int id) {
                this.name = name;
                this.age = age;
                this.id = id;
            }

            @Override
            public boolean equals(Object o) {
                if (this == o) return true;
                if (o == null || getClass() != o.getClass()) return false;
                User user = (User) o;
                // 自定义相等条件：只有 id 一样就相等
                return id == user.id;
            }

            @Override
            public int hashCode() {
                // 自定义哈希值：只使用 id 作为哈希值
                return Objects.hash(id);
            }
        }

        // 创建 User 对象
        User user1 = new User("user1", 18, 1);
        User user2 = new User("user1", 18, 1); // 与 user1 的 id 相同
        User user3 = new User("user3", 19, 2); // 不同 id

        // 创建 Map
        Map<Object, String> map = new HashMap<>();

        // 将键值对放入 Map
        map.put(1, "Value1");
        map.put(user3, "Value3");

        // 验证条件
        System.out.println(map.get(1));          // 输出: Value1
        System.out.println(map.get(user2));      // 输出: Value1 (因为 user2 的 id == 1，与键 1 相等)
        System.out.println(map.get(user3));      // 输出: Value3 (user3 的 id == 2，与键 1 不相等)
    }
}
~~~


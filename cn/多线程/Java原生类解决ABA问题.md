# Java解决ABA问题

Java中解决ABA的问题主要是用的是java.util.concurrent.atomic包中的原子类（AtomicStampedReference、AtomicMarkableReference），用来解决ABA问题。

## AtomicStampedReference

AtomicStampedReference通过引入一个标记的概念来解决ABA问题。他不仅存储了对象的引用，还存储了一个整数的标记值，每次更新引用的时候，标记值爷随之更新。因此，如果预期值不同，标记值不同也会被认为是不同的。

~~~java
import java.util.concurrent.atomic.AtomicStampedReference;

public class ABADemo {
    public static void main(String[] args) {
        String initialRef = "A";
        int initialStamp = 0;
        
        AtomicStampedReference<String> atomicStampedRef = new AtomicStampedReference<>(initialRef, initialStamp);

        // 获取当前的引用和标记
        String reference = atomicStampedRef.getReference();
        int stamp = atomicStampedRef.getStamp();
        System.out.println("Initial Reference: " + reference + ", Initial Stamp: " + stamp);

        // 尝试使用 CAS 更新引用
        boolean isUpdated = atomicStampedRef.compareAndSet(reference, "B", stamp, stamp + 1);
        System.out.println("Updated to B: " + isUpdated + ", New Stamp: " + atomicStampedRef.getStamp());

        // 尝试ABA问题：修改回原来的值
        isUpdated = atomicStampedRef.compareAndSet("B", "A", atomicStampedRef.getStamp(), atomicStampedRef.getStamp() + 1);
        System.out.println("Updated back to A: " + isUpdated + ", New Stamp: " + atomicStampedRef.getStamp());

        // 尝试CAS操作期望的初始值和标记
        isUpdated = atomicStampedRef.compareAndSet(initialRef, "C", initialStamp, initialStamp + 1);
        System.out.println("CAS with Initial Reference and Stamp: " + isUpdated + ", Final Reference: " + atomicStampedRef.getReference());
    }
}
~~~

## AtomicMarkableReference

使用一个布尔标记来追踪对象的状态，防止ABA问题。

他依赖布尔标记来区分状态，当状态经过多次变化，例如从false到true再到false。这样的ABA的问题依然可能出现。复杂的场景还是使用AtomicStampedRefernce.

~~~java
import java.util.concurrent.atomic.AtomicMarkableReference;

public class ABAMarkableDemo {
    public static void main(String[] args) {
        String initialRef = "A";
        boolean initialMark = false;

        AtomicMarkableReference<String> atomicMarkableRef = new AtomicMarkableReference<>(initialRef, initialMark);

        // 获取当前的引用和标记
        boolean[] markHolder = new boolean[1];
        String reference = atomicMarkableRef.get(markHolder);
        System.out.println("Initial Reference: " + reference + ", Initial Mark: " + markHolder[0]);

        // 尝试使用 CAS 更新引用
        boolean isUpdated = atomicMarkableRef.compareAndSet(reference, "B", initialMark, true);
        System.out.println("Updated to B: " + isUpdated + ", New Mark: " + atomicMarkableRef.isMarked());

        // 尝试ABA问题：修改回原来的值
        isUpdated = atomicMarkableRef.compareAndSet("B", "A", true, false);
        System.out.println("Updated back to A: " + isUpdated + ", New Mark: " + atomicMarkableRef.isMarked());

        // 尝试CAS操作期望的初始值和标记
        isUpdated = atomicMarkableRef.compareAndSet(initialRef, "C", initialMark, true);
        System.out.println("CAS with Initial Reference and Mark: " + isUpdated + ", Final Reference: " + atomicMarkableRef.getReference());
    }
}
~~~


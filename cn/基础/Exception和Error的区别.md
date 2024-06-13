# Exception和Error的区别

## Exception

Exception就是可以被预料和通过try-catch处理的异常，比如文件没找到。

~~~java
public class ExceptionExample {
    public static void main(String[] args) {
        try {
            // 可能抛出受检异常
            throw new IOException("IO错误");
        } catch (IOException e) {
            System.out.println("捕获到异常: " + e.getMessage());
        }

        try {
            // 可能抛出未受检异常
            throw new NullPointerException("空指针错误");
        } catch (NullPointerException e) {
            System.out.println("捕获到异常: " + e.getMessage());
        }
    }
}

~~~

### Exception的2大类

### 受检异常Check Excepitons

必须使用try-catch和throws进行处理

## 未受检异常

不需要try-catch或者throws处理，程序可以忽略

~~~java
public class UncheckedExceptionExample {
    public static void main(String[] args) {
        try {
            int[] array = new int[5];
            System.out.println(array[10]); // 引发 ArrayIndexOutOfBoundsException
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("捕获到未受检异常: " + e.getMessage());
        }

        // 不捕获未受检异常，程序仍然可以编译
        String str = null;
        System.out.println(str.length()); // 引发 NullPointerException
    }
}
~~~

## Error

表示是我们程序无法恢复的严重问题，错误通常是程序之外的情况，比如硬件故障，内存不足。比如OutofMemoryError和StackOverflowError、VirtualMachineError等。

~~~java
public class ErrorExample {
    public static void main(String[] args) {
        try {
            // 可能抛出错误，通常不应该捕获
            throw new StackOverflowError("栈溢出错误");
        } catch (StackOverflowError e) {
            System.out.println("捕获到错误: " + e.getMessage());
        }

        // 一般情况下，不捕获错误，程序会因为错误终止
        throw new OutOfMemoryError("内存不足错误");
    }
}

~~~


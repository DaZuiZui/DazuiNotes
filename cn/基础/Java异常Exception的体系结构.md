# Java异常Exception的体系结构

Java异常体系主要是**Throwable**为根，它主要有两个重要的子类。**Error**和**Exception**。

## Error

这个表示系统级别的错误，通常为虚拟机的报告，在我们应用程序无法处理，通常发生在运行时环境问题，如内存不足，堆栈溢出。OutOfMemoryError和StackOverfloError。

## Exception

这种报错都是我们应用程序可以解决的，比如空指针，指针越界等。
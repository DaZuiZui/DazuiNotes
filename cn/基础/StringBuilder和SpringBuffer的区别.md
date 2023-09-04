# StringBuilder和SpringBuffer的区别

## StringBuilder

​	StringBuilder不是线程安全的，这就意味着多个线程对StringBuilder进行访问的时候，可能造成数据不一致或异常。因为它适用于单线程的情况，如果我们确保使用环境在单线程的情况可以使用Stringbuilder

​	StringBuilder是处理可变的字符串，主要用于处理可变字符串。它每次对字符串执行插入、追加、删除和修改操作等操作不会创建新的字符串对象，因此它在处理大量字符串拼接和修改的时候有很高的性能。

**Stringbuilder的特点**	

​	**可变性：**Stringbuilder对象的内容可以随时修改，它可以避免创建新的字符串对象。比较适用频繁修改字符串内容的场景。

​	**性能：** StringBuilder是可变的，它避免了创建新的字符串对象，从而减少了内存分配和垃圾回收的开销， 因此字符串操作性能方面要优于String。

## StringBuffer

StringBuffer的定义和StringBuilder只有一点不同，Stringbuffer是线程安全的， 如果我们使用了synchronized关键字进行了同步，保证了线程安全，如果我们要在多线程环境下进行操作字符串，那就使用Stringbuffer进行操作。
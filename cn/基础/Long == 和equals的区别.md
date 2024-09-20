# Long == 和equals的区别

其实他和Integer是一样的，



==是比较的内存地址，如果是[-128,127]会进行缓存的，JVM会缓存这个范围的对象，

~~~java
Long long1 = Long.valueOf(127);
Long long2 = Long.valueOf(127);

System.out.println(long1 == long2); // true，因为 127 在缓存范围内，long1 和 long2 引用了相同的对象

Long long1 = Long.valueOf(128);
Long long2 = Long.valueOf(128);
System.out.println(long1 == long2); // false，因为 128 不在缓存范围内，long1 和 long2 是不同的对象
~~~

一般来说这些缓存不会被清理除非一些特殊情况，比如JVM关闭，类卸载，内存紧张。





而equals是比较的两个值，是否一样。
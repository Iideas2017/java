---
description: 比较三个java类的区别
---

# String、StringBuilder、StringBuffer区别

        这三个类之间的区别主要是在两个方面，即**运行速度**和**线程安全**这两方面。

## 1. 运行速度

#### （**在这方面运行速度快慢为：StringBuilder &gt; StringBuffer &gt; String**）

             **String最慢的原因：**String为**字符串常量**，而StringBuilder和StringBuffer均为**字符串变量**，即String对象一旦创建之后该对象是**不可更改**的，但后两者的对象是变量，是可以更改的。例如：

```java
String str="abc";
System.out.println(str);
str=str+"de";
System.out.println(str);
```

           JVM对于这几行代码是这样处理的：首先创建一个String对象str，并把“abc”赋值给str，然后在第三行中，其实JVM又创建了一个新的对象也名为str，再把原来的str的值和“de”加起来再赋值给新的str，而原来的str就会被JVM的垃圾回收机制（GC）给回收掉了，所以，str实际上并没有被更改，也就是前面说的String对象一旦创建之后就不可更改了。

         Java中对String对象进行的操作实际上是一个**不断创建新的对象并且将旧的对象回收**的一个过程，所以执行速度很**慢**。                     

        StringBuilder和StringBuffer的对象是变量，对变量进行操作就是**直接对该对象进行更改，而不进行创建和回收的操作**，所以速度要比String**快**很多。

##  **2. 线程安全**

        **在线程安全上，StringBuilder是线程不安全的，而StringBuffer是线程安全的**

　　****如果一个StringBuffer对象在字符串缓冲区被多个线程使用时，StringBuffer中很多方法可以带有**synchronized关键字**，所以可以保证线程是安全的，但StringBuilder的方法则没有该关键字，所以不能保证线程安全，有可能会出现一些错误的操作。所以如果要进行的操作是多线程的，那么就要使用StringBuffer，但是在**单线程**的情况下，还是建议使用**速度比较快**的StringBuilder。

## 3. 总结

        **String：适用于少量的字符串操作的情况**

　　**StringBuilder：适用于单线程下在字符缓冲区进行大量操作的情况**

　　**StringBuffer：适用多线程下在字符缓冲区进行大量操作的情况**

## **4. 参考**

\*\*\*\*[**https://blog.csdn.net/fairy\_huang/article/details/76070200**](https://blog.csdn.net/fairy_huang/article/details/76070200)\*\*\*\*


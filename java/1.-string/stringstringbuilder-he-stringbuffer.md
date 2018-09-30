---
description: 运行速度快慢为：StringBuilder > StringBuffer > String
---

# 2.5 String、StringBuilder 和 StringBuffer

## **1. String 与 StringBuilder**

　　简要的说， **String 类型 和 StringBuilder 类型的主要性能区别在于 String 是不可变的对象。** 事实上，在对 String 类型进行“改变”时，实质上等同于生成了一个新的 String 对象，然后将指针指向新的 String 对象。由于频繁的生成对象会对系统性能产生影响，特别是当内存中没有引用指向的对象多了以后，JVM 的垃圾回收器就会开始工作，继而会影响到程序的执行效率。所以，**对于经常改变内容的字符串，最好不要声明为 String 类型**。但如果我们使用的是 StringBuilder 类，那么情形就不一样了。因为，我们的每次修改都是针对 StringBuilder 对象本身的，而不会像对String操作那样去生成新的对象并重新给变量引用赋值。所以，**在一般情况下，推荐使用 StringBuilder ，特别是字符串对象经常改变的情况下**。

　　**在某些特别情况下，String 对象的字符串拼接可以直接被JVM 在编译期确定下来，这时，StringBuilder 在速度上就不占任何优势了。**

　　因此，在绝大部分情况下， 在效率方面：StringBuilder &gt; String .

## **2. StringBuffer 与 StringBuilder**

　　首先需要明确的是，StringBuffer 始于 JDK 1.0，而 StringBuilder 始于 JDK 5.0；此外，从 JDK 1.5 开始，对含有字符串变量 \(非字符串字面值\) 的连接操作\(+\)，**JVM 内部是采用 StringBuilder 来实现的**，而在这之前，这个操作是采用 StringBuffer 实现的。

　　**JDK的实现中 StringBuffer 与 StringBuilder 都继承自 AbstractStringBuilder。**

AbstractStringBuilder的实现原理为：AbstractStringBuilder中采用一个 **char数组** 来保存需要append的字符串，char数组有一个初始大小，当append的字符串长度超过当前char数组容量时，则**对char数组进行动态扩展**，即重新申请一段更大的内存空间，然后将当前char数组拷贝到新的位置，因为重新分配内存并拷贝的开销比较大，所以每次重新申请内存空间都是采用申请大于当前需要的内存空间的方式，这里是 **2** 倍。

　　**StringBuffer 和 StringBuilder 都是可变的字符序列，但是二者最大的一个不同点是：StringBuffer 是线程安全的，而 StringBuilder 则不是。StringBuilder 提供的API与StringBuffer的API是完全兼容的，即，StringBuffer 与 StringBuilder 中的方法和功能完全是等价的，但是后者一般要比前者快。因此，可以这么说，StringBuilder 的提出就是为了在单线程环境下替换 StringBuffer 。**

　　在单线程环境下，优先使用 StringBuilder。

## **3. 实例**

### 1\). 编译时优化与字符串连接符的本质

　　我们先来看下面这个例子：

```java
public class Test2 {
    public static void main(String[] args) {
        String s = "a" + "b" + "c";
        String s1 = "a";
        String s2 = "b";
        String s3 = "c";
        String s4 = s1 + s2 + s3;

        System.out.println(s);
        System.out.println(s4);
    }
}
```

　　由上面的叙述，我们可以知道，变量s的创建等价于 String s = “abc”; 

       而变量s4的创建相当于：

```java
    StringBuilder temp = new StringBuilder(s1);
    temp.append(s2).append(s3);
    String s4 = temp.toString();123
```

　　但事实上，是不是这样子呢？我们将其反编译一下，来看看Java编译器究竟做了什么：

```java
//将上述 Test2 的 class 文件反编译
public class Test2
{
    public Test2(){}
    public static void main(String args[])
    {
        String s = "abc";            // 编译期优化
        String s1 = "a";
        String s2 = "b";
        String s3 = "c";

        //底层使用 StringBuilder 进行字符串的拼接
        String s4 = (new StringBuilder(String.valueOf(s1))).append(s2).append(s3).toString();   
        System.out.println(s);
        System.out.println(s4);
    }
}
```

　　根据上面的反编译结果，很好的印证了字符串连接符的本质。

### 2\). 另一个例子：字符串连接符的本质

　　由上面的分析结果，我们不难推断出 String 采用连接运算符（+）效率低下原因分析，形如这样的代码：

```java
public class Test { 
    public static void main(String args[]) { 
        String s = null; 
            for(int i = 0; i < 100; i++) { 
                s += "a"; 
            } 
    }
}12345678
```

会被编译器编译为：

```java
public class Test
{
    public Test(){}
    public static void main(String args[])
    {
        String s = null;
        for (int i = 0; i < 100; i++)
            s = (new StringBuilder(String.valueOf(s))).append("a").toString();
    }
}
```

　　也就是说，每做一次 字符串连接操作 “+” 就产生一个 StringBuilder 对象，然后 append 后就扔掉。下次循环再到达时，再重新 new 一个 StringBuilder 对象，然后 append 字符串，如此循环直至结束。事实上，如果我们直接采用 StringBuilder 对象进行 append 的话，我们可以节省 **N - 1** 次创建和销毁对象的时间。所以，**对于在循环中要进行字符串连接的应用，一般都是用StringBulider对象来进行append操作。**

## 4. 总结

        **String：适用于少量的字符串操作的情况**

　　**StringBuilder：适用于单线程下在字符缓冲区进行大量操作的情况**

　　**StringBuffer：适用多线程下在字符缓冲区进行大量操作的情况**


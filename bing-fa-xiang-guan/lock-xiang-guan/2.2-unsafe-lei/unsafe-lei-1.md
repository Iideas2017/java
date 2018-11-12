# 2.2.2 作用

## 一、**综述**

在Java9中，为了提高JVM的可维护性，Unsafe和许多其他的东西一起都被作为内部使用类隐藏起来了。

### **1、做一些Java语言不允许但是又十分有用的事情**

很多低级语言中可用的技巧在Java中都是不被允许的。对大多数开发者而言这是件好事，既可以拯救你，也可以拯救你的同事们。同样也使得导入开源代码更容易了，因为你能掌握它们可以造成的最大的灾难上限。或者至少明确你可以不小心失误的界限。如果你尝试地足够努力，你也能造成损害。

那你可能会奇怪，为什么还要去尝试呢？当建立库时，Unsafe中很多（但不是所有）方法都很有用，且有些情况下，除了使用JNI，没有其他方法做同样的事情，即使它可能会更加危险同时也会失去Java的“一次编译，永久运行”的跨平台特性。

### **2、对象的反序列化**

当使用框架反序列化或者构建对象时，会假设从已存在的对象中重建，你期望使用反射来调用类的设置函数，或者更准确一点是能直接设置**内部字段甚至是final字段的函数**。问题是你想创建一个对象的实例，但你实际上又不需要构造函数，因为它可能会使问题更加困难而且会有副作用。

```java
package com.jvm.study.unsafe;

import java.io.Serializable;

public class A implements Serializable {
    private final int num;

    public A(int num) {
        System.out.println("Hello Mum");
        this.num = num;
    }

    public int getNum() {
        return num;
    }
}
```

在这个类中，应该能够重建和设置final字段，但如果你不得不调用构造函数时，它就可能做一些和反序列化无关的事情。有了这些原因，很多库使用Unsafe创建实例而不是调用构造函数。

```java
    public static Unsafe getUnsafe() {
        try {
            Field f = Unsafe.class.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            return (Unsafe) f.get(null);
        } catch (Exception e) {
            return null;
        }
    }

    public static void test2() throws InstantiationException {
        Unsafe unsafe = getUnsafe();
        Class aClass = A.class;
        A a = (A) unsafe.allocateInstance(aClass);
        System.out.println("test2():"+a.getNum());   //test2():0
    }
```

调用allocateInstance函数避免了在我们不需要构造函数的时候却调用它。

### **3、线程安全的直接获取内存**

**Unsafe的另外一个用途是线程安全的获取非堆内存**。ByteBuffer函数也能使你安全的**获取非堆内存或是DirectMemory，但它不会提供任何线程安全的操作**。你在**进程间共享数据时**使用Unsafe尤其有用。

```java
package com.jvm.study.unsafe;

import sun.misc.Unsafe;
import sun.nio.ch.DirectBuffer;

import java.io.File;
import java.io.IOException;
import java.io.RandomAccessFile;
import java.lang.reflect.Field;
import java.nio.MappedByteBuffer;
import java.nio.channels.FileChannel;

public class PingPongMapMain {
    public static void main(String[] args) throws IOException {
        //test("odd");
    }
    public static void test(String... args) throws IOException {
        boolean odd;
        switch (args.length < 1 ? "usage" : args[0].toLowerCase()) {
        case "odd":
            odd = true;
            break;
        case "even":
            odd = false;
            break;
        default:
            System.err.println("Usage: java PingPongMain [odd|even]");
            return;
        }
        int runs = 10000000;
        long start = 0;
        System.out.println("Waiting for the other odd/even");
        File counters = new File(System.getProperty("java.io.tmpdir"), "counters.deleteme");
        counters.deleteOnExit();

        try (FileChannel fc = new RandomAccessFile(counters, "rw").getChannel()) {
            MappedByteBuffer mbb = fc.map(FileChannel.MapMode.READ_WRITE, 0, 1024);
            long address = ((DirectBuffer) mbb).address();
            for (int i = -1; i < runs; i++) {
                for (;;) {
                    long value = UNSAFE.getLongVolatile(null, address);
                    boolean isOdd = (value & 1) != 0;
                    if (isOdd != odd)
                        // wait for the other side.
                        continue;
                    // make the change atomic, just in case there is more than
                    // one odd/even process
                    if (UNSAFE.compareAndSwapLong(null, address, value, value + 1))
                        break;
                }
                if (i == 0) {
                    System.out.println("Started");
                    start = System.nanoTime();
                }
            }
        }
        System.out.printf("... Finished, average ping/pong took %,d ns%n", (System.nanoTime() - start) / runs);
    }

    static final Unsafe UNSAFE;

    static {
        try {
            Field theUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
            theUnsafe.setAccessible(true);
            UNSAFE = (Unsafe) theUnsafe.get(null);
        } catch (Exception e) {
            throw new AssertionError(e);
        }
    }
}
```

当你分别在两个程序，一个输入odd一个输入even，中运行时，可以看到两个进程都是通过持久化共享内存交换数据的。

在每个程序中，将相同的磁盘缓存映射到进程中。内存中实际上只有一份文件的副本存在。这意味着内存可以共享，前提是你使用线程安全的操作，比如volatile变量和CAS操作。（译注：CAS Compare and Swap 无锁算法）

在两个进程之间有83ns的往返时间。当考虑到System V IPC（进程间通信）大约需要2500ns，而且用IPC volatile替代persisted内存，算是相当快的了。

### **4、Unsafe适合在工作中使用吗？**

个人不建议直接使用Unsafe。它远比原生的Java开发所需要的测试多。基于这个原因建议还是使用经过测试的库。如果你只是想自己用Unsafe，建议你最好在一个独立的类库中进行全面的测试。这限制了Unsafe在你的应用程序中的使用方式，但会给你一个更安全的Unsafe。

### **5、总结**

Unsafe在Java中是很有趣的一个存在，你可以一个人在家里随便玩玩。它也有一些工作的应用程序特别是在写底层库的时候，但总的来说，使用经过测试的Unsafe库比直接用要好。


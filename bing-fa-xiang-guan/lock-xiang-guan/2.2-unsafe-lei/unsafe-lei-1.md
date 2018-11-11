# 2.2.2 Unsafe 类\(下）

## 一、**综述**

在Java9中，为了提高JVM的可维护性，Unsafe和许多其他的东西一起都被作为内部使用类隐藏起来了。

### **1、做一些Java语言不允许但是又十分有用的事情**

很多低级语言中可用的技巧在Java中都是不被允许的。对大多数开发者而言这是件好事，既可以拯救你，也可以拯救你的同事们。同样也使得导入开源代码更容易了，因为你能掌握它们可以造成的最大的灾难上限。或者至少明确你可以不小心失误的界限。如果你尝试地足够努力，你也能造成损害。

那你可能会奇怪，为什么还要去尝试呢？当建立库时，Unsafe中很多（但不是所有）方法都很有用，且有些情况下，除了使用JNI，没有其他方法做同样的事情，即使它可能会更加危险同时也会失去Java的“一次编译，永久运行”的跨平台特性。

### **2、对象的反序列化**

当使用框架反序列化或者构建对象时，会假设从已存在的对象中重建，你期望使用反射来调用类的设置函数，或者更准确一点是能直接设置内部字段甚至是final字段的函数。问题是你想创建一个对象的实例，但你实际上又不需要构造函数，因为它可能会使问题更加困难而且会有副作用。

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

**Unsafe的另外一个用途是线程安全的获取非堆内存**。ByteBuffer函数也能使你安全的获取非堆内存或是DirectMemory，但它不会提供任何线程安全的操作。你在进程间共享数据时使用Unsafe尤其有用。

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

## 二、使用Unsafe操作内存中的Java类和对象

**让我们开始展示内存中Java类和对象结构**

你可曾好奇过Java内存管理核心构件？你是否问过自己某些奇怪的问题，比如：

* 一个类在内存中占据多少空间？
* 我的对象在内存中消耗了多少空间？
* 对象的属性在内存中是如何被布局的？

如果这些问题听起来很熟悉，那么你就想到了点子上。对于像我们这样的在RebelLabs的Java极客来说，这些难解的谜题已经在我们脑海中缠绕了很长时间：如果你对探究类检测器感兴趣，想知道如何布局让所有的类更容易地从内存中取到指定变量，或是想在系统运行时侵入内存中的这些字段。这就意味着你能切实改变内存中的数据甚至是代码！

其它可能勾起你兴趣的知识点有，“堆外缓存”和“高性能序列化”的实现。这是一对构建在对象缓存结构上很好的实例，揭示了获取类和实例内存地址的方法，缓存中类和实例的布局以及关于对象成员变量布局的详细解释。我们希望尽可能简单地阐释这些内容，但是尽管如此这篇文章并不适合Java初学者，它要求具备对Java编程原理有一定的了解。

**注意：下面关于类和对象的布局所写的内容特指Java SE 7**，所以不推荐使用者想当然地认为这些适用于过去或将来的Java版本。

**在Java中最直接的内存操作方法是什么？**

Java最初被设计为一种安全的受控环境。尽管如此，Java HotSpot还是包含了一个“后门”，提供了一些可以直接操控内存和线程的低层次操作。这个后门类——sun.misc.Unsafe——被JDK广泛用于自己的包中，如java.nio和java.util.concurrent。但是丝毫不建议在生产环境中使用这个后门。因为这个API十分不安全、不轻便、而且不稳定。这个不安全的类提供了一个观察HotSpot JVM内部结构并且可以对其进行修改。有时它可以被用来在不适用C++调试的情况下学习虚拟机内部结构，有时也可以被拿来做性能监控和开发工具。

**为何变得不安全**

sun.misc.Unsafe这个类是如此地不安全，以至于JDK开发者增加了很多特殊限制来访问它。它的构造器是私有的，工厂方法getUnsafe\(\)的调用器只能被Bootloader加载。如你在下面代码片段的第8行所见，这个家伙甚至没有被任何类加载器加载，所以它的类加载器是null。它会抛出SecurityException 异常来阻止侵入者。

```java
public final class Unsafe {
   ...
   private Unsafe() {}
   private static final Unsafe theUnsafe = new Unsafe();
   ...
   public static Unsafe getUnsafe() {
      Class cc = sun.reflect.Reflection.getCallerClass(2);
      if (cc.getClassLoader() != null)
          throw new SecurityException("Unsafe");
      return theUnsafe;
   }
   ...
}
```

幸运的是这里有一个Unsafe的变量可以被用来取得Unsafe的实例。

```java
public static Unsafe getUnsafe() {
   try {
           Field f = Unsafe.class.getDeclaredField("theUnsafe");
           f.setAccessible(true);
           return (Unsafe)f.get(null);
   } catch (Exception e) {
       /* ... */
   }
}
```



**Unsafe一些有用的特性**

1. 虚拟机“集约化”（VM intrinsification）：如用于无锁Hash表中的CAS（比较和交换）。再比如compareAndSwapInt这个方法用JNI调用，包含了对CAS有特殊引导的本地代码。在这里你能读到更多关于CAS的信息：[http://en.wikipedia.org/wiki/Compare-and-swap](http://en.wikipedia.org/wiki/Compare-and-swap)。
2. 主机虚拟机（_译注：主机虚拟机主要用来管理其他虚拟机。而虚拟平台我们看到只有guest VM_）的sun.misc.Unsafe功能能够被用于未初始化的对象分配内存（用allocateInstance方法），然后将构造器调用解释为其他方法的调用。
3. 你可以从本地内存地址中追踪到这些数据。使用java.lang.Unsafe类获取内存地址是可能的。而且可以通过unsafe方法直接操作这些变量！
4. 使用allocateMemory方法，内存可以被分配到堆外。例如当allocateDirect方法被调用时DirectByteBuffer构造器内部会使用allocateMemory。
5. arrayBaseOffset和arrayIndexScale方法可以被用于开发arraylets，一种用来将大数组分解为小对象、限制扫描的实时消耗或者在大对象上做更新和移动。

**普通对象指针和压缩普通对象指针**

堆中的Java对象使用普通对象指针（OOP）来表示的。OOP是指向Java堆内部某一内存地址的管理指针，在JVM处理器的虚地址空间中是一块单独的连续地址区域。

OOP通常和机器指针大小相同，在一个LP64的系统中就是64比特。在一个ILP32系统中，堆最大尺寸比40亿字节稍微少一点，足以满足大多数应用了。

_译注：32位环境涉及”ILP32″数据模型，是因为C数据类型为32位的int、long、指针。而64位环境使用不同的数据模型，此时的long和指针已为64位，故称作”LP64″数据模型。_

Java堆中的管理指针指向8字节对齐对象（[https://wikis.oracle.com/display/HotSpotInternals/CompressedOops ](https://wikis.oracle.com/display/HotSpotInternals/CompressedOops)）。在大多数情况下，压缩普通对象指针代表了JVM中从64比特的堆基地址偏移32比特的对象管理指针。因为它们都是对象偏移而不是字节偏移，所以可以用于寻址40亿对象或者320亿字节的堆大小。使用时，必须乘以8并且加上Java堆的基地址去定位对象的位置。使用压缩普通对象指针的对象大小和ILP32系统差不多。

在Java v6u23及以后的版本中，支持并且默认启用压缩普通对象指针。在Java v7中，当没有指定”-Xmx”时，64比特  
JVM处理器默认使用压缩普通对象指针；指定”-Xmx”时小于320亿字节。在6u23版本之前的JDK 6，在Java命令中使用”-XX:+UseCompressedOops”标志启用这一特性（[http://docs.oracle.com/javase/7/docs/technotes/guides/vm/performance-enhancements-7.html ](http://docs.oracle.com/javase/7/docs/technotes/guides/vm/performance-enhancements-7.html)）。

在我们的示例中，通过“-XX:-UseCompressedOops” 关闭了压缩普通对象指针功能，所以64bit的指针大小就是8字节。

**关于示例类的一些事**

在这篇文章中，我们将使用一个示例类（SampleClass）展示对象地址恢复、列出字段布局等。这是一个简单类，包含了三个基本数据类型并且继承了SampleBaseClass，用来展示继承的内存布局。示例类的定义如下，示例代码可以在GitHub上找到：

```java
package com.jvm.study.unsafe;

public class SampleBaseClass {
    protected short s = 20;
}
package com.jvm.study.unsafe;

public final class SampleClass extends SampleBaseClass {

    private final static byte b = 100;

    private int i = 5;
    private long l = 10;

    public SampleClass() {

    }

    public SampleClass(int i, long l) {
        this.i = i;
        this.l = l;
    }

    public int getI() {
        return i;
    }

    public void setI(int i) {
        this.i = i;
    }

    public long getL() {
        return l;
    }

    public void setL(long l) {
        this.l = l;
    }

    public static byte getB() {
        return b;
    }
}
```

要得到Java类的内存地址没有简便方法。为了得到地址，必须使用一些技巧并且做一些牺牲！本文会介绍两种获得Java类内存地址的办法。

**方法一**

在JVM中，每个对象都一个指向类的指针。但是只指向具体类，不支持接口或抽象类。如果我们得到一个对象的内存地址，就可以很容易地找到类的地址。这种方法对于那些可以创建实例的类来说非常有用。但是接口或抽象类不能使用这种方法。

```text
For 32 bit JVM:
	_mark	: 4 byte constant
	_klass	: 4 byte pointer to class 

For 64 bit JVM:
	_mark	: 8 byte constant
	_klass	: 8 byte pointer to class

For 64 bit JVM with compressed-oops:
	_mark	: 8 byte constant
	_klass	: 4 byte pointer to class
```

内存中对象的第二个字段（对32位JVM偏移是4，64位JVM偏移是8）指向了内存中的类定义。你可以使用_“sun.misc.Unsafe”_ 类得到此偏移的内存值。这里用到的是在上一篇中提到的`SampleClass`。

```text
For 32 bit JVM:
	SampleClass sampleClassObject = new SampleClass();
	int addressOfSampleClass = unsafe.getInt(sampleClassObject, 4L);

For 64 bit JVM:
	SampleClass sampleClassObject = new SampleClass();
	long addressOfSampleClass = unsafe.getLong(sampleClassObject, 8L);

For 64 bit JVM with compressed-oops:
	SampleClass sampleClassObject = new SampleClass();
	long addressOfSampleClass = unsafe.getInt(sampleClassObject, 8L);
```

**方法2**

使用这种方法，可以得到任何类的内存地址（包括接口、注解、抽象类和枚举）。Java7中类定义的内存地址结构如下：32位JVM的地址偏移从第4到80字节，64位JVM的地址偏移从8字节到160字节，压缩普通对象指针的地址偏移从第4到84字节。

没有预先定义好的偏移，但是在类文件解析器中作为“隐藏”字段给出了注释（这里实际上有3个字段：`class`, `arrayClass`, `resolvedConstructor`）。因为在java.lang.Class中有18个非静态引用字段，他们只是恰好表示了这段偏移。

更多信息可以参见`ClassFileParser::java_lang_Class_fix_pre()` `和JavaClasses::check_offsets()`。

获取内存地址的示例代码如下：

```text
For 32 bit JVM:
	int addressOfSampleClass = unsafe.getInt(SampleClass.class, 80L);

For 64 bit JVM:
	long addressOfSampleClass = unsafe.getLong(SampleClass.class, 160L);

For 64 bit JVM with compressed-oops:
	long addressOfSampleClass = unsafe.getInt(SampleClass.class, 84L);

```

**怎样才能获得对象内存地址？**

获取对象内存地址要比获取类内存地址更加需要技巧。我们需要使用长度和 java.lang.Object 类型的辅助数组（长度为1）获得对象的内存地址。

下面是获取对象内存地址的详细步骤：

1、将目标对象设为辅助数组的第一个元素（也是唯一的元素）。由于这是一个复杂类型元素（不是基本数据类型），它的地址存储在数组的第一个元素。

2、然后，获取辅助数组的基本偏移量。数组的基本偏移量是指数组对象的起始地址与数组第一个元素之间的偏移量。

3、确定JVM的地址空间：

* 如果是32位JVM，可以通过 sun.misc.Unsafe 类得到数组对象的内存地址（address\_of\_array）与数组基本偏移量（base\_offset\_of\_array）相加的整型结果。这个4字节整型数值就是目标对象的内存地址。
* 如果是64位JVM，可以通过 sun.misc.Unsafe 类得到数组对象的内存地址（address\_of\_array）与数组基本偏移量（base\_offset\_of\_array）相加的长整型值结果。这个8字节长整型数值就是目标对象的内存地址。

32位JVM：

```java
Object helperArray[]    = new Object[1];
helperArray[0]      = targetObject;
long baseOffset     = unsafe.arrayBaseOffset(Object[].class);
int addressOfObject = unsafe.getInt(helperArray, baseOffset);
```

64位JVM：

```java
Object helperArray[]    = new Object[1];
helperArray[0]      = targetObject;
long baseOffset     = unsafe.arrayBaseOffset(Object[].class);
long addressOfObject    = unsafe.getLong(helperArray, baseOffset);
```

可以认为这段代码中的 targetObject 是上文中 SampleClass 的某个实例。但请记住，这段代码适用于任何类的任何实例。

**类的内存布局**

32位JVM：

```text
[header                ] 4  byte
[klass pointer         ] 4  byte (pointer)
[C++ vtbl ptr          ] 4  byte (pointer)
[layout_helper         ] 4  byte
[super check offset    ] 4  byte 
[name                  ] 4  byte (pointer)
[secondary super cache ] 4  byte (pointer)
[secondary supers      ] 4  byte (pointer)
[primary supers        ] 32 byte (8 length array of pointer)
[java mirror           ] 4  byte (pointer)
[super                 ] 4  byte (pointer)
[first subklass        ] 4  byte (pointer)
[next sibling          ] 4  byte (pointer)
[modifier flags        ] 4  byte
 4  byte
```

64位JVM：

```text
[header                ] 8  byte
[klass pointer         ] 8  byte (4 byte for compressed-oops)
[C++ vtbl ptr          ] 8  byte (4 byte for compressed-oops)
[layout_helper         ] 4  byte
[super check offset    ] 4  byte 
[name                  ] 8  byte (4 byte for compressed-oops)
[secondary super cache ] 8  byte (4 byte for compressed-oops)
[secondary supers      ] 8  byte (4 byte for compressed-oops)
[primary supers        ] 64 byte (32 byte for compressed-oops)
                                     {8 length array of pointer}
[java mirror           ] 8  byte (4 byte for compressed-oops)
[super                ] 8  byte (4 byte for compressed-oops)
[first subklass         ] 8  byte (4 byte for compressed-oops)
[next sibling          ] 8  byte (4 byte for compressed-oops)
[modifier flags        ] 4  byte
 4  byte
```



下图展示了 SampleClass 在32位JVM中的内存布局，列出了自起始地址起的前128个字节：

[![](http://www.importnew.com/wp-content/uploads/2014/01/java-memory-layout-1.jpg)](http://www.importnew.com/8494.html/java-memory-layout-1)

[![](http://www.importnew.com/wp-content/uploads/2014/01/java-memory-layout-2.jpg)](http://www.importnew.com/8494.html/java-memory-layout-2)

这是 SampleBaseClass 在32位JVM中的内存布局，列出了自起始地址起的前128个字节：

[![](http://www.importnew.com/wp-content/uploads/2014/01/java-memory-layout-3.jpg)](http://www.importnew.com/8494.html/java-memory-layout-3)

我们只对重要的字段进行说明，数字的颜色对应着相应字段的颜色。

header 是一个常量：0×00000001。

**klass pointer** 指向 java.lang.Class 类在内存中的定义（即 java.lang.Class 类的内存地址，这两个类的klass pointer字段都指向0x38970v8a8），表明这是一个类的内存结构。

C++ vtbl ptr 指向其对应类的虚函数表。虚函数表用于实现运行时虚函数调用的多态性。

layout helper 用于记录该实例的浅尺寸（Shallow size）。浅尺寸是根据JVM的字段对齐机制计算出来的。在我们的环境中，对象按8字节对齐。

**super** 指向其父类的定义，在我们的演示代码中，SampleBaseClass 是 SampleClass 的父类。在 SampleClass 类的内存布局中，可以看到SampleBaseClass 的内存地址为0x34104b70。而在 SampleBaseClass 类的内存布局中， 父类的内存地址为0×38970000，这是  java.lang.Object 类的地址。因为在Java中，每一个类都是 Object 类的子类。

modifier flags 即类的修饰符标志位。在Java中类的修饰符包括：public、protected、private、abstract、static、final以及strictfp。modifier flags 字段的值是对目标类的所有修饰符进行按位或运算得到的。在我们的示例代码中，SampleClass 类的修饰符有 public 和 final 。因此它的 modifier flags 字段的值为“0×00000001 \| 0×00000010 =  0×00000011 ”。而 SampleBaseClass 类的修饰符只有 public，所以它的modifier flags 字段的值为0×00000001 。各修饰符的对应取值如下所示：

[![](http://www.importnew.com/wp-content/uploads/2014/01/values.png)](http://www.importnew.com/8494.html/values)

**字段布局和对齐**

和C/C++不同，Java没有 sizeOf 运算符计算基本数据类型类型或对象所占用的内存空间，sizeOf 运算符在IO操作和内存管理中非常实用。事实上，由于基本数据类型的大小在语言规范中预先定义，Java中也不会出现指针拷贝内存和指针运算（因为没有指针）。因此，sizeOf 运算符并没有存在的必要。

有两种方法能够确定一个类及其属性共占用了多少内存空间。分别是浅尺寸（shallow size）和深尺寸（deep size）。浅尺寸就是对象本身各个字段所占用内存的大小，不包含该类中所引用的其他对象；于是引入了深尺寸概念，深尺寸在浅尺寸的基础上增加了该类引用的其他对象的浅尺寸。

Sun Java虚拟机规定：除数组外的所有对象都包含一个长度为双字（two-word，一个字由两个字节组成）的 header，一个长度为单字（one-word）flags，以及一个指向对应类引用的单字（one-word）的字段。当我们用 new Object\(\) 方法创建一个对象时，堆上就会为其分配8个字节的内存空间。

对于一个继承了 Object 的类来说，情况会变得复杂而有趣。在这8个字节后面，类的各个属性在堆内存上会按照一定的规则对齐，但并不按照他们的声明顺序进行对齐。

基本数据类型按照下列顺序进行对齐：

* double、long
* int、float
* short、char
* boolean、byte

接下来，该类中所引用其他类的对象也会在堆上进行对齐。JVM会把对象的大小调整为8字节的倍数（[http://www.codeinstructions.com/2008/12/java-objects-memory-structure.html](http://www.codeinstructions.com/2008/12/java-objects-memory-structure.html)）。

请看下面的示例：

| 123 | `class` `BooleanClass {byte` `a;}` |
| :--- | :--- |


这里会自动填充7个字节，整个对象的大小被扩大到16个字节。

| 123 | `Headers (include flags and ref to` `class) :` `8` `bytesvalue of` `byte` `:` `1` `bytepadding :` `7` `bytes` |
| :--- | :--- |


**关于OOP的更多信息**

有关OOP的一些基本信息已经在 OOP 和 压缩OOP 章节介绍过了。我们假定你已经对JVM中 OOP 相关的术语有一定的了解了，下面就让我们对它作进一步的了解。

OOP由两个机器字长度的字段组成（32位JVM上机器字长为4字节，64位JVM上机器字长为8字节），这两个字段分别是 Mark 和 Klass。这两个字段出现在该实例的所有成员字段之前（_译注：这两个字段应该就是对应上面字段的布局和对齐章节中 headers 所占用的8个字节_）。但对于数组对象来说，在这两个字段之前会有一个额外的字段用于记录数组的长度。Mark 字段用于垃圾回收（在mark-and-sweep回收算法中使用），Klass 字段用于指向其对应类的元数据。所有的基本数据类型字段和引用字段都排在OOP（Mark 和 Klass 字段）的后面，包括引用其他对象的字段，甚至是引用OOP的字段（ [http://www.infoq.com/articles/Introduction-to-HotSpot](http://www.infoq.com/articles/Introduction-to-HotSpot) ）。

**KlassOOPs**

Klass 字段是一个指向对应类的元数据（包括字段的定义以及类似C++的虚函数表）的指针。每一个实例都携带一份类的元数据是一种非常低效的方式，KlassOOPs能让所有对象共享同一份元数据从而减少不必要的开销。需要注意的是 KlassOOP 和类加载器所产生的 Class object（java.lang.class 类型的对象）并不相同。下面是两者之间的区别：

* **Class objects** 只是普通的Java对象。Class objects和其他的Java对象一样可以用OOP（InstanceOOPs）表示。其表现行为也与其他的Java对象相同，还可以存放到Java变量里。
* **KlassOOPs** 是类的元数据在JVM中的表现形式，例如类的虚函数表就存放在 KlassOOPs 中。由于 KlassOOPs 生存在堆的永久区里（Permgen space），因此在Java代码中无法直接获得 KlassOOPs 的引用。你也可以简单地认为 KlassOOP 是对应类的 Class object 在虚拟机级别上的镜像。

**MarkOOPs**

Mark字段指向一个维护着OOP相关管理信息的数据结构。在32位JVM中，mark字段的数据结构为（[http://hg.openjdk.java.net/jdk7/hotspot/hotspot/file/9b0ca45cd756/src/share/vm/oops/markOop.hpp for more details ](http://hg.openjdk.java.net/jdk7/hotspot/hotspot/file/9b0ca45cd756/src/share/vm/oops/markOop.hpp%20for%20more%20details)）：

* **哈希值（25 bits）：**记录着该对象的 HashCode\(\) 方法的返回值。
* **年龄（4 bits）：**对象的年龄（即这个对象所经历过的垃圾回收的次数）。
* **偏向锁（1 bit）+ 锁（2 bits）：**用于表示该对象的同步状态。

Java 5引入了一种全新的对象同步方式，叫做偏向锁（Java 6中默认使用偏向锁 Biased-Lock）。经过观察发现，在多数情况下对象在运行时往往只被一个线程锁住，因此引入了偏向锁的概念。处于偏向锁状态中的对象会优先朝向第一个锁住它的线程，这个线程也会获得到更好的锁性能。Mark字段中会记录获取到该对象偏向锁的线程：

* **Java线程指针：**23 bits
* **历元时间戳（Epoch）：**2 bits
* **年龄：**4 bits
* **偏向锁状态：**1 bit
* **锁状态：**2 bits

如果另一个线程尝试锁定该对象，偏向锁就会失效（无法重新获取）。这样，所有的线程都必须通过显式调用lock、unlock方法来锁定和解锁对象。

下面是对象可能出现的状态：

* 未锁定状态（Unlocked）
* 偏向锁状态（Biased）
* 轻量级锁定（Lightweight Locked）
* 重量级锁定（Heavyweight Locked）
* 标记状态（Marked，仅会在垃圾回收期间出现）

32位JVM

| 123 | `[mark]         ] 8  byte[klass pointer ] 8  byte (pointer)[fields        ] values of all fields including fields from super classe` |
| :--- | :--- |


64位JVM

| 123 | `[mark]         ] 8  byte[klass pointer ] 8  byte (4 byte for compressed-oops)[fields        ] values of all fields including fields from super classes` |
| :--- | :--- |


请参考：[http://hg.openjdk.java.net/jdk7/hotspot/hotspot/file/9b0ca45cd756/src/share/vm/oops/oop.hpp](http://hg.openjdk.java.net/jdk7/hotspot/hotspot/file/9b0ca45cd756/src/share/vm/oops/oop.hpp)。

下面我们来聊一聊深尺寸的计算，并考虑继承关系的影响。我们继续在32位的JVM上以 SampleClass 和 SampleBaseClass 为例。下面是 SampleClass 对象的内存布局。请再仔细看看这两个类的代码和各个字段，以便于更好的理解后续内容。

[![](http://www.importnew.com/wp-content/uploads/2014/01/java-memory-layout-4.jpg)](http://www.importnew.com/8514.html/java-memory-layout-4)  
mark字段是内存布局中的第一个字（0x69e34e01），字段中包含有该对象的哈希值，还有锁状态、对象年龄之类的标志位。

klass 字段指向 SampleClass 类的定义，即0x34104cc0。

字段 s 的值是20（0×0014），s是父类 SampleBaseClass 中的字段。父类字段排在内存布局的最前面，并且不会和子类的字段交叉排列。内存布局中的父类字段会以完整的系统字长结束。在字段的结尾，如果字长为4字节会按4字节进行自动补齐；如果字长为8字节，也会按4字节进行自动补齐。两个填充字节（0×0000）可用于填充长度为4字节（一个字长）的空隙。

字段 i 的值为5（0×00000005）。字段 **l** 排在字段 i 的后面，它的值为10（**0x000000000000000a**）。

正如上文提到的类属性顺序：首先是long和double，其次是int和float，然后是char和short，再然后是byte和boolean，最后是引用类型。属性按照各自的粒度进行对齐。当子类的第一个字段是double或者long类型，而父类并不满足8字节对齐，JVM为了填补这个空隙会破例尝试在子类的最前面摆放一个相对较短的字段。JVM会依次尝试摆放int、short和byte类型的字段，最后尝试引用类型的字段。

因此整型字段排在长整型字段的前面，字段 i 排在字段 l 的前面。

**本文到此结束，我们还会带来更多的惊喜！**

我们希望你喜欢本文对Java中非常酷的底层机制所做的深入探讨，希望能从中有所收获。现在你已经知道了如何利用不安全的后门直接访问内存和线程并完成一些底层操作，能够通过一个类的对象轻松的获取到这个类和这个实例的内存地址，或者是根据预定义的偏移量计算出类的内存地址。

现在你还知道如何了解一个类的内存布局，知道如何通过字段的完整对齐来最小化内存的占用。本文通篇都以 SampleClass 类为例，在保持示例一致的同时也有助于读者更好的理解本文的内容。我们还详细介绍了所有32位JVM和64位JVM相关的例子，希望本文能覆盖到更多的读者。


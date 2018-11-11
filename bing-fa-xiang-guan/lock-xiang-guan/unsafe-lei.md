# Unsafe 类

## 一、Unsafe简介

　　sun.misc.Unsafe类型从名字看，这个类应该是封装了一些不安全的操作。

1、可以用来**在任意内存地址位置处读写数据**，可见，对于普通用户来说，使用起来还是比较危险的；

2、还支持一些CAS原子操作；

　　简单讲一下这个类。**Java无法直接访问底层操作系统，而是通过本地（native）方法来访问**。**不过尽管如此，JVM还是开了一个后门，JDK中有一个类Unsafe，它提供了硬件级别的原子操作**。

这个类尽管里面的方法都是public的，但是并没有办法使用它们，JDK API文档也没有提供任何关于这个类的方法的解释。总而言之，对于Unsafe类的使用都是受限制的，只有授信的代码才能获得该类的实例，当然JDK库里面的类是可以随意使用的。

从第一行的描述可以了解到Unsafe提供了硬件级别的操作，比如说获取某个属性在内存中的位置，比如说修改对象的字段值，即使它是私有的。不过Java本身就是为了屏蔽底层的差异，对于一般的开发而言也很少会有这样的需求。

举两个例子，比方说：

```java
public native long staticFieldOffset(Field paramField);
```

这个方法可以用来获取给定的paramField的内存地址偏移量，这个值对于给定的field是唯一的且是固定不变的。

## 二、Unsafe源码分析

### 2.1、类图结构

#### 2.2、数据结构

#### 2.2、Unsafe中的lock

#### 2.3、成员变量

#### 2.3、构造函数

```java
    private Unsafe()
    {}

    public static Unsafe getUnsafe()
    {
        Class class1 = Reflection.getCallerClass();
        if(!VM.isSystemDomainLoader(class1.getClassLoader()))
            throw new SecurityException("Unsafe");
        else
            return theUnsafe;
    }
```

\*不能直接`new Unsafe()`，原因是`Unsafe`被设计成单例模式，构造方法是私有的；

\*不能通过调用`Unsafe.getUnsafe()获取，因为getUnsafe`被设计成只能从引导类加载器（bootstrap class loader）加载，从`getUnsafe`的源码中也可以看出来；

JDK的开发人员并不希望大家使用这个类。获得Unsafe实例的方法是只能调用其工厂方法getUnsafe\(\)。上面其中的if\(!VM.isSystemDomainLoader\(class1.getClassLoader\(\)\)\)的判断是为了只有JDK内部类调用。其它应用层调用它都报错。

**示例1：**

通过下面的例子可以测试：

```java
package com.dxz.unsafe;
public class Test {

    public static void main(String[] args) {
        sun.misc.Unsafe u = sun.misc.Unsafe.getUnsafe();
        System.out.println(u);
    }
}
```

在if\(!VM.isSystemDomainLoader\(class1.getClassLoader\(\)\)\)处打个断点，然后watch下Reflection.getCallerClass\(\)的值，如下图：

![](../../.gitbook/assets/image%20%2847%29.png)

        ****说明：根据Java 类加载器的工作原理，应用程序的类由AppLoader加载。而系统核心类，如rt.jar中的类由Bootstrap类加载器加载。Bootstrap加载器没有Java对象的对象，因此试图获得这个类加载器会返回null。所以，当一个类的类加载器为null时，说明它是由Bootstrap加载的，而这个类也极有可能是rt.jar中的类。

## 三、Unsafe的原子操作

Unsafe类提供了硬件级别的原子操作，主要提供了以下功能：

### 1、分配、释放内存

通过Unsafe类可以分配内存，可以释放内存；

类中提供的3个本地方法**allocateMemory**、**reallocateMemory**、**freeMemory**分别用于分配内存，扩充内存和释放内存，与C语言中的3个方法对应。

```java
//分配内存
public native long allocateMemory(long l);
//扩充内存
public native long reallocateMemory(long l, long l1);
//释放内存
public native void freeMemory(long l);
```

### 2、定位内存

#### 可以定位对象某字段的内存位置，也可以修改对象的字段值，即使它是私有的；

**1 字段的定位：**

JAVA中对象的字段的定位可能通过native staticFieldOffset方法实现，该方法返回给定field的内存地址偏移量，这个值对于给定的filed是唯一的且是固定不变的。  
native getIntVolatile方法获取对象中offset偏移地址对应的整型field的值,支持volatile load语义。  
native getLong方法获取对象中offset偏移地址对应的long型field的值。

2 **数组元素定位：**

Unsafe类中有很多以BASE\_OFFSET结尾的常量，比如ARRAY\_INT\_BASE\_OFFSET，ARRAY\_BYTE\_BASE\_OFFSET等，这些常量值是通过arrayBaseOffset方法得到的。arrayBaseOffset方法是一个本地方法，可以获取数组第一个元素的偏移地址。Unsafe类中还有很多以INDEX\_SCALE结尾的常量，比如 ARRAY\_INT\_INDEX\_SCALE ， ARRAY\_BYTE\_INDEX\_SCALE等，这些常量值是通过arrayIndexScale方法得到的。arrayIndexScale方法也是一个本地方法，可以获取数组的转换因子，也就是数组中元素的增量地址。将arrayBaseOffset与arrayIndexScale配合使用，可以定位数组中每个元素在内存中的位置。

```java
public final class Unsafe {
    public static final int ARRAY_INT_BASE_OFFSET;
    public static final int ARRAY_INT_INDEX_SCALE;

    public native long staticFieldOffset(Field field);
    public native int getIntVolatile(Object obj, long l);
    public native long getLong(Object obj, long l);
    public native int arrayBaseOffset(Class class1);
    public native int arrayIndexScale(Class class1);

    static 
    {
        ARRAY_INT_BASE_OFFSET = theUnsafe.arrayBaseOffset([I);
        ARRAY_INT_INDEX_SCALE = theUnsafe.arrayIndexScale([I);
    }
}
```

### 3、挂起与恢复

将一个线程进行挂起是通过park方法实现的，调用park后，线程将一直阻塞直到超时或者中断等条件出现。unpark可以终止一个挂起的线程，使其恢复正常。整个并发框架中对线程的挂起操作被封装在LockSupport类中，LockSupport类中有各种版本pack方法，但最终都调用了Unsafe.park\(\)方法。

```java
public class LockSupport {
    public static void unpark(Thread thread) {
        if (thread != null)
            unsafe.unpark(thread);
    }

    public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        unsafe.park(false, 0L);
        setBlocker(t, null);
    }

    public static void parkNanos(Object blocker, long nanos) {
        if (nanos > 0) {
            Thread t = Thread.currentThread();
            setBlocker(t, blocker);
            unsafe.park(false, nanos);
            setBlocker(t, null);
        }
    }

    public static void parkUntil(Object blocker, long deadline) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        unsafe.park(true, deadline);
        setBlocker(t, null);
    }

    public static void park() {
        unsafe.park(false, 0L);
    }

    public static void parkNanos(long nanos) {
        if (nanos > 0)
            unsafe.park(false, nanos);
    }

    public static void parkUntil(long deadline) {
        unsafe.park(true, deadline);
    }
}
```

上面调用的Unsafe中的park和unpark的native方法如下：

```java
    public native void unpark(Object obj);

    public native void park(boolean flag, long l);
```

### 4、CAS操作

是通过compareAndSwapXXX方法实现的

```java
/**
* 比较obj的offset处内存位置中的值和期望的值，如果相同则更新。此更新是不可中断的。
* 
* @param obj 需要更新的对象
* @param offset obj中整型field的偏移量
* @param expect 希望field中存在的值
* @param update 如果期望值expect与field的当前值相同，设置filed的值为这个新值
* @return 如果field的值被更改返回true
*/
public native boolean compareAndSwapInt(Object obj, long offset, int expect, int update);
```

CAS操作有3个操作数，内存值M，预期值E，新值U，如果M==E，则将内存值修改为B，否则啥都不做。

看下natUnsafe.cc中的c++实现吧，加深理解，其实就是将内存值与预期值作比较，判断是否相等，相等的话，写入数据，不相等不做操作，返回旧数据；

```java
static inline bool
compareAndSwap (volatile jint *addr, jint old, jint new_val)
{
  jboolean result = false;
  spinlock lock;
  if ((result = (*addr == old)))
    *addr = new_val;
  return result;
}
```

### 5、**直接修改内存**

#### putLong，putInt，putDouble，putChar，putObject等方法，直接修改内存数据（可以越过访问权限）

这里，还有put对应的get方法，很简单就是直接读取内存地址处的数据，不做举例；

 我们可以举个putLong\(Object, long, long\)方法详细看下其具体实现，其它的类似，先看Java的源码，没啥好看的，就声明了一个native本地方法：

三个参数说明下：

```java
Object o//对象引用
long offset//对象内存地址的偏移量
long x//写入的数据
```

```java
public native void  putLong(Object o, long offset, long x);
```

还是看下natUnsafe.cc中的c++实现吧，很简单，就是计算要写入数据的内存地址，然后写入数据，如下：

```java
void
sun::misc::Unsafe::putLong (jobject obj, jlong offset, jlong value)
{
  jlong *addr = (jlong *) ((char *) obj + offset);//计算要修改的数据的内存地址=对象地址+成员属性地址偏移量
  spinlock lock;//自旋锁，通过循环来获取锁， i386处理器需要加锁访问64位数据，如果是int，则不需要改行代码
  *addr = value;//往该内存地址位置直接写入数据
}
```

**示例2：**

即使User类的成员属性是私有的且没有提供对外的public方法，我们还是可以直接在它们的内存地址位置处写入数据，并成功；

```java
package com.dxz.unsafe;

import java.lang.reflect.Field;

import sun.misc.Unsafe;

class User {
    private String name = "test";
    private long id = 1;
    private int age = 2;
    private double height = 1.72;

    @Override
    public String toString() {
        return name + "," + id + "," + age + "," + height;
    }
}

public class Test2 {
    public static void main(String[] args) throws NoSuchFieldException, SecurityException, IllegalArgumentException,
            IllegalAccessException, InstantiationException {
        // 通过反射得到theUnsafe对应的Field对象
        Field field = Unsafe.class.getDeclaredField("theUnsafe");
        // 设置该Field为可访问
        field.setAccessible(true);
        // 通过Field得到该Field对应的具体对象，传入null是因为该Field为static的
        Unsafe unsafe = (Unsafe) field.get(null);

        User user = new User();
        System.out.println(user); // 打印test,1,2,1.72

        Class userClass = user.getClass();
        Field name = userClass.getDeclaredField("name");
        Field id = userClass.getDeclaredField("id");
        Field age = userClass.getDeclaredField("age");
        Field height = userClass.getDeclaredField("height");
        // 直接往内存地址写数据
        unsafe.putObject(user, unsafe.objectFieldOffset(name), "midified-name");
        unsafe.putLong(user, unsafe.objectFieldOffset(id), 100l);
        unsafe.putInt(user, unsafe.objectFieldOffset(age), 101);
        unsafe.putDouble(user, unsafe.objectFieldOffset(height), 100.1);

        System.out.println(user);// 打印midified-name,100,101,100.1

    }
}
```

结果：

```text
test,1,2,1.72
midified-name,100,101,100.1
```

### 6、**操作Volatile变量**

getLongVolatile/putLongVolatile等等方法

这类方法使用volatile语义去存取数据，我的理解就是各个线程不缓存数据，直接在内存中读取数据；

## 四、JDK或开源框架中使用

LockSupport中park阻塞，和unpark解除阻塞。  
concurrent中中的cas调用。

## 五、源码

Unsafe源码：

```java
//下面是sun.misc.Unsafe.java类源码
package sun.misc;
import java.lang.reflect.Field;
/***
 * This class should provide access to low-level operations and its
 * use should be limited to trusted code.  Fields can be accessed using
 * memory addresses, with undefined behaviour occurring if invalid memory
 * addresses are given.
 * 这个类提供了一个更底层的操作并且应该在受信任的代码中使用。可以通过内存地址
 * 存取fields,如果给出的内存地址是无效的那么会有一个不确定的运行表现。
 * 
 * @author Tom Tromey (tromey@redhat.com)
 * @author Andrew John Hughes (gnu_andrew@member.fsf.org)
 */
public class Unsafe
{
  // Singleton class.
  private static Unsafe unsafe = new Unsafe();
  /***
   * Private default constructor to prevent creation of an arbitrary
   * number of instances.
   * 使用私有默认构造器防止创建多个实例
   */
  private Unsafe()
  {
  }
  /***
   * Retrieve the singleton instance of <code>Unsafe</code>.  The calling
   * method should guard this instance from untrusted code, as it provides
   * access to low-level operations such as direct memory access.
   * 获取<code>Unsafe</code>的单例,这个方法调用应该防止在不可信的代码中实例，
   * 因为unsafe类提供了一个低级别的操作，例如直接内存存取。
   * 
   * @throws SecurityException if a security manager exists and prevents
   *                           access to the system properties.
   *                           如果安全管理器不存在或者禁止访问系统属性
   */
  public static Unsafe getUnsafe()
  {
    SecurityManager sm = System.getSecurityManager();
    if (sm != null)
      sm.checkPropertiesAccess();
    return unsafe;
  }
  
  /***
   * Returns the memory address offset of the given static field.
   * The offset is merely used as a means to access a particular field
   * in the other methods of this class.  The value is unique to the given
   * field and the same value should be returned on each subsequent call.
   * 返回指定静态field的内存地址偏移量,在这个类的其他方法中这个值只是被用作一个访问
   * 特定field的一个方式。这个值对于 给定的field是唯一的，并且后续对该方法的调用都应该
   * 返回相同的值。
   *
   * @param field the field whose offset should be returned.
   *              需要返回偏移量的field
   * @return the offset of the given field.
   *         指定field的偏移量
   */
  public native long objectFieldOffset(Field field);
  /***
   * Compares the value of the integer field at the specified offset
   * in the supplied object with the given expected value, and updates
   * it if they match.  The operation of this method should be atomic,
   * thus providing an uninterruptible way of updating an integer field.
   * 在obj的offset位置比较integer field和期望的值，如果相同则更新。这个方法
   * 的操作应该是原子的，因此提供了一种不可中断的方式更新integer field。
   * 
   * @param obj the object containing the field to modify.
   *            包含要修改field的对象
   * @param offset the offset of the integer field within <code>obj</code>.
   *               <code>obj</code>中整型field的偏移量
   * @param expect the expected value of the field.
   *               希望field中存在的值
   * @param update the new value of the field if it equals <code>expect</code>.
   *           如果期望值expect与field的当前值相同，设置filed的值为这个新值
   * @return true if the field was changed.
   *                             如果field的值被更改
   */
  public native boolean compareAndSwapInt(Object obj, long offset,
                                          int expect, int update);
  /***
   * Compares the value of the long field at the specified offset
   * in the supplied object with the given expected value, and updates
   * it if they match.  The operation of this method should be atomic,
   * thus providing an uninterruptible way of updating a long field.
   * 在obj的offset位置比较long field和期望的值，如果相同则更新。这个方法
   * 的操作应该是原子的，因此提供了一种不可中断的方式更新long field。
   * 
   * @param obj the object containing the field to modify.
   *              包含要修改field的对象 
   * @param offset the offset of the long field within <code>obj</code>.
   *               <code>obj</code>中long型field的偏移量
   * @param expect the expected value of the field.
   *               希望field中存在的值
   * @param update the new value of the field if it equals <code>expect</code>.
   *               如果期望值expect与field的当前值相同，设置filed的值为这个新值
   * @return true if the field was changed.
   *              如果field的值被更改
   */
  public native boolean compareAndSwapLong(Object obj, long offset,
                                           long expect, long update);
  /***
   * Compares the value of the object field at the specified offset
   * in the supplied object with the given expected value, and updates
   * it if they match.  The operation of this method should be atomic,
   * thus providing an uninterruptible way of updating an object field.
   * 在obj的offset位置比较object field和期望的值，如果相同则更新。这个方法
   * 的操作应该是原子的，因此提供了一种不可中断的方式更新object field。
   * 
   * @param obj the object containing the field to modify.
   *    包含要修改field的对象 
   * @param offset the offset of the object field within <code>obj</code>.
   *         <code>obj</code>中object型field的偏移量
   * @param expect the expected value of the field.
   *               希望field中存在的值
   * @param update the new value of the field if it equals <code>expect</code>.
   *               如果期望值expect与field的当前值相同，设置filed的值为这个新值
   * @return true if the field was changed.
   *              如果field的值被更改
   */
  public native boolean compareAndSwapObject(Object obj, long offset,
                                             Object expect, Object update);
  /***
   * Sets the value of the integer field at the specified offset in the
   * supplied object to the given value.  This is an ordered or lazy
   * version of <code>putIntVolatile(Object,long,int)</code>, which
   * doesn't guarantee the immediate visibility of the change to other
   * threads.  It is only really useful where the integer field is
   * <code>volatile</code>, and is thus expected to change unexpectedly.
   * 设置obj对象中offset偏移地址对应的整型field的值为指定值。这是一个有序或者
   * 有延迟的<code>putIntVolatile</cdoe>方法，并且不保证值的改变被其他线程立
   * 即看到。只有在field被<code>volatile</code>修饰并且期望被意外修改的时候
   * 使用才有用。
   * 
   * @param obj the object containing the field to modify.
   *    包含需要修改field的对象
   * @param offset the offset of the integer field within <code>obj</code>.
   *       <code>obj</code>中整型field的偏移量
   * @param value the new value of the field.
   *      field将被设置的新值
   * @see #putIntVolatile(Object,long,int)
   */
  public native void putOrderedInt(Object obj, long offset, int value);
  /***
   * Sets the value of the long field at the specified offset in the
   * supplied object to the given value.  This is an ordered or lazy
   * version of <code>putLongVolatile(Object,long,long)</code>, which
   * doesn't guarantee the immediate visibility of the change to other
   * threads.  It is only really useful where the long field is
   * <code>volatile</code>, and is thus expected to change unexpectedly.
   * 设置obj对象中offset偏移地址对应的long型field的值为指定值。这是一个有序或者
   * 有延迟的<code>putLongVolatile</cdoe>方法，并且不保证值的改变被其他线程立
   * 即看到。只有在field被<code>volatile</code>修饰并且期望被意外修改的时候
   * 使用才有用。
   * 
   * @param obj the object containing the field to modify.
   *    包含需要修改field的对象
   * @param offset the offset of the long field within <code>obj</code>.
   *       <code>obj</code>中long型field的偏移量
   * @param value the new value of the field.
   *      field将被设置的新值
   * @see #putLongVolatile(Object,long,long)
   */
  public native void putOrderedLong(Object obj, long offset, long value);
  /***
   * Sets the value of the object field at the specified offset in the
   * supplied object to the given value.  This is an ordered or lazy
   * version of <code>putObjectVolatile(Object,long,Object)</code>, which
   * doesn't guarantee the immediate visibility of the change to other
   * threads.  It is only really useful where the object field is
   * <code>volatile</code>, and is thus expected to change unexpectedly.
   * 设置obj对象中offset偏移地址对应的object型field的值为指定值。这是一个有序或者
   * 有延迟的<code>putObjectVolatile</cdoe>方法，并且不保证值的改变被其他线程立
   * 即看到。只有在field被<code>volatile</code>修饰并且期望被意外修改的时候
   * 使用才有用。
   *
   * @param obj the object containing the field to modify.
   *    包含需要修改field的对象
   * @param offset the offset of the object field within <code>obj</code>.
   *       <code>obj</code>中long型field的偏移量
   * @param value the new value of the field.
   *      field将被设置的新值
   */
  public native void putOrderedObject(Object obj, long offset, Object value);
  /***
   * Sets the value of the integer field at the specified offset in the
   * supplied object to the given value, with volatile store semantics.
   * 设置obj对象中offset偏移地址对应的整型field的值为指定值。支持volatile store语义
   * 
   * @param obj the object containing the field to modify.
   *    包含需要修改field的对象
   * @param offset the offset of the integer field within <code>obj</code>.
   *       <code>obj</code>中整型field的偏移量
   * @param value the new value of the field.
   *       field将被设置的新值
   */
  public native void putIntVolatile(Object obj, long offset, int value);
  /***
   * Retrieves the value of the integer field at the specified offset in the
   * supplied object with volatile load semantics.
   * 获取obj对象中offset偏移地址对应的整型field的值,支持volatile load语义。
   * 
   * @param obj the object containing the field to read.
   *    包含需要去读取的field的对象
   * @param offset the offset of the integer field within <code>obj</code>.
   *       <code>obj</code>中整型field的偏移量
   */
  public native int getIntVolatile(Object obj, long offset);
  /***
   * Sets the value of the long field at the specified offset in the
   * supplied object to the given value, with volatile store semantics.
   * 设置obj对象中offset偏移地址对应的long型field的值为指定值。支持volatile store语义
   *
   * @param obj the object containing the field to modify.
   *            包含需要修改field的对象
   * @param offset the offset of the long field within <code>obj</code>.
   *               <code>obj</code>中long型field的偏移量
   * @param value the new value of the field.
   *              field将被设置的新值
   * @see #putLong(Object,long,long)
   */
  public native void putLongVolatile(Object obj, long offset, long value);
  /***
   * Sets the value of the long field at the specified offset in the
   * supplied object to the given value.
   * 设置obj对象中offset偏移地址对应的long型field的值为指定值。
   * 
   * @param obj the object containing the field to modify.
   *     包含需要修改field的对象
   * @param offset the offset of the long field within <code>obj</code>.
   *     <code>obj</code>中long型field的偏移量
   * @param value the new value of the field.
   *     field将被设置的新值
   * @see #putLongVolatile(Object,long,long)
   */
  public native void putLong(Object obj, long offset, long value);
  /***
   * Retrieves the value of the long field at the specified offset in the
   * supplied object with volatile load semantics.
   * 获取obj对象中offset偏移地址对应的long型field的值,支持volatile load语义。
   * 
   * @param obj the object containing the field to read.
   *    包含需要去读取的field的对象
   * @param offset the offset of the long field within <code>obj</code>.
   *       <code>obj</code>中long型field的偏移量
   * @see #getLong(Object,long)
   */
  public native long getLongVolatile(Object obj, long offset);
  /***
   * Retrieves the value of the long field at the specified offset in the
   * supplied object.
   * 获取obj对象中offset偏移地址对应的long型field的值
   * 
   * @param obj the object containing the field to read.
   *    包含需要去读取的field的对象
   * @param offset the offset of the long field within <code>obj</code>.
   *       <code>obj</code>中long型field的偏移量
   * @see #getLongVolatile(Object,long)
   */
  public native long getLong(Object obj, long offset);
  /***
   * Sets the value of the object field at the specified offset in the
   * supplied object to the given value, with volatile store semantics.
   * 设置obj对象中offset偏移地址对应的object型field的值为指定值。支持volatile store语义
   * 
   * @param obj the object containing the field to modify.
   *    包含需要修改field的对象
   * @param offset the offset of the object field within <code>obj</code>.
   *     <code>obj</code>中object型field的偏移量
   * @param value the new value of the field.
   *       field将被设置的新值
   * @see #putObject(Object,long,Object)
   */
  public native void putObjectVolatile(Object obj, long offset, Object value);
  /***
   * Sets the value of the object field at the specified offset in the
   * supplied object to the given value.
   * 设置obj对象中offset偏移地址对应的object型field的值为指定值。
   * 
   * @param obj the object containing the field to modify.
   *    包含需要修改field的对象
   * @param offset the offset of the object field within <code>obj</code>.
   *     <code>obj</code>中object型field的偏移量
   * @param value the new value of the field.
   *       field将被设置的新值
   * @see #putObjectVolatile(Object,long,Object)
   */
  public native void putObject(Object obj, long offset, Object value);
  /***
   * Retrieves the value of the object field at the specified offset in the
   * supplied object with volatile load semantics.
   * 获取obj对象中offset偏移地址对应的object型field的值,支持volatile load语义。
   * 
   * @param obj the object containing the field to read.
   *    包含需要去读取的field的对象
   * @param offset the offset of the object field within <code>obj</code>.
   *       <code>obj</code>中object型field的偏移量
   */
  public native Object getObjectVolatile(Object obj, long offset);
  /***
   * Returns the offset of the first element for a given array class.
   * To access elements of the array class, this value may be used along with
   * with that returned by 
   * <a href="#arrayIndexScale"><code>arrayIndexScale</code></a>,
   * if non-zero.
   * 获取给定数组中第一个元素的偏移地址。
   * 为了存取数组中的元素，这个偏移地址与<a href="#arrayIndexScale"><code>arrayIndexScale
   * </code></a>方法的非0返回值一起被使用。
   * @param arrayClass the class for which the first element's address should
   *                   be obtained.
   *                   第一个元素地址被获取的class
   * @return the offset of the first element of the array class.
   *    数组第一个元素 的偏移地址
   * @see arrayIndexScale(Class)
   */
  public native int arrayBaseOffset(Class arrayClass);
  /***
   * Returns the scale factor used for addressing elements of the supplied
   * array class.  Where a suitable scale factor can not be returned (e.g.
   * for primitive types), zero should be returned.  The returned value
   * can be used with 
   * <a href="#arrayBaseOffset"><code>arrayBaseOffset</code></a>
   * to access elements of the class.
   * 获取用户给定数组寻址的换算因子.一个合适的换算因子不能返回的时候(例如：基本类型),
   * 返回0.这个返回值能够与<a href="#arrayBaseOffset"><code>arrayBaseOffset</code>
   * </a>一起使用去存取这个数组class中的元素
   * 
   * @param arrayClass the class whose scale factor should be returned.
   * @return the scale factor, or zero if not supported for this array class.
   */
  public native int arrayIndexScale(Class arrayClass);
  
  /***
   * Releases the block on a thread created by 
   * <a href="#park"><code>park</code></a>.  This method can also be used
   * to terminate a blockage caused by a prior call to <code>park</code>.
   * This operation is unsafe, as the thread must be guaranteed to be
   * live.  This is true of Java, but not native code.
   * 释放被<a href="#park"><code>park</code></a>创建的在一个线程上的阻塞.这个
   * 方法也可以被使用来终止一个先前调用<code>park</code>导致的阻塞.
   * 这个操作操作时不安全的,因此线程必须保证是活的.这是java代码不是native代码。
   * @param thread the thread to unblock.
   *           要解除阻塞的线程
   */
  public native void unpark(Thread thread);
  /***
   * Blocks the thread until a matching 
   * <a href="#unpark"><code>unpark</code></a> occurs, the thread is
   * interrupted or the optional timeout expires.  If an <code>unpark</code>
   * call has already occurred, this also counts.  A timeout value of zero
   * is defined as no timeout.  When <code>isAbsolute</code> is
   * <code>true</code>, the timeout is in milliseconds relative to the
   * epoch.  Otherwise, the value is the number of nanoseconds which must
   * occur before timeout.  This call may also return spuriously (i.e.
   * for no apparent reason).
   * 阻塞一个线程直到<a href="#unpark"><code>unpark</code></a>出现、线程
   * 被中断或者timeout时间到期。如果一个<code>unpark</code>调用已经出现了，
   * 这里只计数。timeout为0表示永不过期.当<code>isAbsolute</code>为true时，
   * timeout是相对于新纪元之后的毫秒。否则这个值就是超时前的纳秒数。这个方法执行时
   * 也可能不合理地返回(没有具体原因)
   * 
   * @param isAbsolute true if the timeout is specified in milliseconds from
   *                   the epoch.
   *                   如果为true timeout的值是一个相对于新纪元之后的毫秒数
   * @param time either the number of nanoseconds to wait, or a time in
   *             milliseconds from the epoch to wait for.
   *             可以是一个要等待的纳秒数，或者是一个相对于新纪元之后的毫秒数直到
   *             到达这个时间点
   */
  public native void park(boolean isAbsolute, long time);
}
```

## 六、示例

下面这个例子演示了简单的修改一个byte\[\]的数据。

这个例子在eclipse里不能直接编译，要到项目的属性，Java Compiler，Errors/Warnings中Forbidden reference\(access rules\)中设置为warning。

另外，因为sun.misc.Unsafe包不能直接使用，所有代码里用反射的技巧得到了一个Unsafe的实例。

```java
package com.dxz.unsafe;

import java.lang.reflect.Field;
import java.util.Arrays;
import sun.misc.Unsafe;

public class Test3 {
    private static int byteArrayBaseOffset;

    public static void main(String[] args)
            throws SecurityException, NoSuchFieldException, IllegalArgumentException, IllegalAccessException {
        // 通过反射得到theUnsafe对应的Field对象
        Field theUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
        // 设置该Field为可访问
        theUnsafe.setAccessible(true);
        // 通过Field得到该Field对应的具体对象，传入null是因为该Field为static的
        Unsafe UNSAFE = (Unsafe) theUnsafe.get(null);
        System.out.println(UNSAFE);

        byte[] data = new byte[10];
        System.out.println(Arrays.toString(data));
        //arrayBaseOffset方法是一个本地方法，可以获取数组第一个元素的偏移地址
        byteArrayBaseOffset = UNSAFE.arrayBaseOffset(byte[].class);

        System.out.println(byteArrayBaseOffset);
        //putByte直接修改内存数据
        UNSAFE.putByte(data, byteArrayBaseOffset, (byte) 1);
        UNSAFE.putByte(data, byteArrayBaseOffset + 5, (byte) 5);
        System.out.println(Arrays.toString(data));
    }
}
```

运行结果：

```text
sun.misc.Unsafe@15db9742
[0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
16
[1, 0, 0, 0, 0, 5, 0, 0, 0, 0]
```


# 2.3 String 对象创建方式

## **1. 字面值形式**

        ****JVM会自动根据字符串**常量池中字符串的实际情况**来决定是否创建新对象

        **\(要么不创建，要么创建一个对象，关键要看常量池中有没有\)**

JDK 中明确指出：

```java
String s = "abc";
```

等价于：

```java
char data[] = {'a', 'b', 'c'};
String str = new String(data);
```

　　该种方式先在栈中创建一个对String类的对象引用变量s，然后去查找 “abc”是否被保存在字符串常量池中。**若”abc”已经被保存在字符串常量池中，则在字符串常量池中找到值为”abc”的对象，然后将s 指向这个对象**; 否则，**在堆**中创建char数组 data，然后在**堆**中创建一个String对象object，它由 data 数组组成，紧接着这个String对象 object 被存放进字符串常量池，最后将 s 指向这个对象。

例如：

```java
    private static void test01(){  
    String s0 = "kvill";        // 1
    String s1 = "kvill";        // 2
    String s2 = "kv" + "ill";     // 3

    System.out.println(s0 == s1);       // true  
    System.out.println(s0 == s2);       // true  
}
```

　　执行第 1 行代码时，“kvill” 入池并被 s0 指向；执行第 2 行代码时，s1 从常量池查询到” kvill” 对象并直接指向它；所以，s0 和 s1 指向同一对象。 由于 ”kv” 和 ”ill” 都是字符串字面值，所以 s2 在编译期由编译器直接解析为 “kvill”，所以 s2 也是常量池中”kvill”的一个引用。 所以，我们得出 s0==s1==s2;

## **2. 通过 new 创建字符串对象** 

        **一概在堆中创建新对象，无论字符串字面值是否相等 \(要么创建一个，要么创建两个对象，关键要看常量池中有没有\)**

```java
String s = new String("abc");  
```

等价于：

```java
1、String original = "abc"; 
2、String s = new String(original);
```

　　**通过 new 操作产生一个字符串（“abc”）时，会先去常量池中查找是否有“abc”对象，如果没有，则创建一个此字符串对象并放入常量池中。然后，在堆中再创建“abc”对象，并返回该对象的地址。**

       所以，**对于 String str=new String\(“abc”\)**：**如果常量池中原来没有”abc”，则会产生两个对象（一个在常量池中，一个在堆中）；否则，产生一个对象。**   
　   
　　用 new String\(\) 创建的字符串对象位于堆中，而不是常量池中。它们有自己独立的地址空间，例如，

```java
    private static void test02(){  
    String s0 = "kvill";  
    String s1 = new String("kvill");  
    String s2 = "kv" + new String("ill");  

    String s = "ill";
    String s3 = "kv" + s;    


    System.out.println(s0 == s1);       // false  
    System.out.println(s0 == s2);       // false  
    System.out.println(s1 == s2);       // false  
    System.out.println(s0 == s3);       // false  
    System.out.println(s1 == s3);       // false  
    System.out.println(s2 == s3);       // false  
}  
```

　　例子中，s0 还是常量池中”kvill”的引用，s1 指向运行时创建的新对象”kvill”，二者指向不同的对象。对于s2，因为后半部分是 new String\(“ill”\)，所以无法在编译期确定，在运行期会 new 一个 StringBuilder 对象， 并由 StringBuilder 的 append 方法连接并调用其 toString 方法返回一个新的 “kvill” 对象。此外，s3 的情形与 s2 一样，均含有编译期无法确定的元素。因此，以上四个 “kvill” 对象互不相同。StringBuilder 的 toString 为：

```java
 public String toString() {
    return new String(value, 0, count);   // new 的方式创建字符串
    }
```

**构造函数 String\(String original\) 的源码为：**

```java
    /**
     * 根据源字符串的底层数组长度与该字符串本身长度是否相等决定是否共用支撑数组
     */
    public String(String original) {
        int size = original.count;
        char[] originalValue = original.value;
        char[] v;
        if (originalValue.length > size) {
            // The array representing the String is bigger than the new
            // String itself. Perhaps this constructor is being called
            // in order to trim the baggage, so make a copy of the array.
            int off = original.offset;
            v = Arrays.copyOfRange(originalValue, off, off + size);  // 创建新数组并赋给 v
        } else {
            // The array representing the String is the same
            // size as the String, so no point in making a copy.
            v = originalValue;
        }

        this.offset = 0;
        this.count = size;
        this.value = v;
    }
```

　　由源码可以知道，**所创建的对象在大多数情形下会与源字符串 original 共享 char数组 。**但是，**什么情况下不会共享呢？**   
　　   
　　Take a look at **substring** , and you’ll see how this can happen.

　　Take for instance String s1 = “Abcd”; String s2 = s1.substring\(3\). Here s2.size\(\) is 1, but s2.value.length is 4. **This is because s1.value is the same as s2.value. This is done of performance reasons \(substring is running in O\(1\), since it doesn’t need to copy the content of the original String\).**

```java
String s1 = "Abcd";       // s1 的value为Abcd的数组，offset为 0，count为 4
String s2 = a.substring(3);      // s2 的value也为Abcd的数组，offset为 3，count为 1
String c = new String(s2);      // s2.value.length 为 4，而 original.count = size = 1, 即 s2.value.length > size 成立123
```

　　**Using substring can lead to a memory leak.** Say you have a really long String, and you only want to keep a small part of it. If you just use substring, you will actually keep the original string content in memory. **Doing String snippet = new String\(reallyLongString.substring\(x,y\)\)** , **prevents you from wasting memory backing a large char array no longer needed.**


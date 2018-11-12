# 2.1 String 定义与基础

## 1. String 的声明

![](../../.gitbook/assets/image%20%28435%29.png)

由 JDK 中关于String的声明可以知道：

* **不同字符串可能共享同一个底层char数组**，例如字符串 String s=”abc” 与 s.substring\(1\) 就共享同一个char数组：char\[\] c = {‘a’,’b’,’c’}。其中，前者的 offset 和 count 的值分别为0和3，后者的 offset 和 count 的值分别为1和2。
* offset 和 count 两个成员变量不是多余的，比如，在执行substring操作时。

## 2、 JDK中关于 String 的描述

　The **String** class represents character strings. All **string literals\(字符串字面值\)** in Java programs, such as “abc”, are implemented as instances of this class. **Strings are constant\(常量\); their values cannot be changed after they are created.** **String buffers【StringBuilder OR StringBuffer】 support mutable strings.** **Because String objects are immutable, they can be shared \(** [**享元模式**](http://blog.csdn.net/justloveyou_/article/details/55045638) **\)**.

## 3、 String 类所内置的操作

　　The class **String** includes methods for examining individual characters of the sequence ，for examining individual characters of the sequence, for comparing strings , for searching strings , for extracting substrings and for creating a copy of a string with all characters translated to uppercase or to lowercase. Case mapping is based on the Unicode Standard version specified by the java.lang.Character class.

## 4、“+” 和 toString\(\).

字符串串联符号（”+”）以及将其他对象转换为字符串的特殊支持

　　The Java language provides special support for the **string concatenation operator \(+\)**, and for conversion of other objects to strings. String concatenation is implemented through the **StringBuilder\(JDK1.5 以后\) OR StringBuffer\(JDK1.5 以前\)** class and **its append method**. String conversions\(转化为字符串\) are implemented through the method **toString**, defined by class Object and inherited by all classes in Java.

## 5、注意：

         String不属于八种基本数据类型，String 的实例是一个对象。因为对象的默认值是null，所以String的默认值也是null；但它又是一种特殊的对象，有其它对象没有的一些特性（String 的不可变性导致其像八种基本类型一样，比如，作为方法参数时，像**基本类型的传值效果一样**）。 例如，以下代码片段：

```java
public class StringTest {

    public static void changeStr(String str) {
        String s = str;
        str += "welcome";
        System.out.println(s);
    }

    public static void main(String[] args) {
        String str = "1234";
        changeStr(str);
        System.out.println(str);
    }
}/* Output: 
        1234
        1234 
*///:
```

* new String\(\) 和 new String\(“”\)都是声明一个**新的空字符串**，是空串不是null;


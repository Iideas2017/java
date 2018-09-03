# String 类

###  1. 没有使用new关键字

```java
String s1="abc1";//此处是数字1
String s2="abc"+1;
System.out.println(s1==s2);// 第一次比较 true
String s3="ab";
String s4="c";
String s5="abc";
String s6=s3+s4;
System.out.println(s5==s6);// 第二次比较 false
```

        s6运行时是这样的：

**`String s6=new StringBuilder().append(s3).append(s4).toString();`**

       这里的过程是通过StringBuilder这个类实现的，我们来看一下StringBuilder类中的toString\(\)的源码：

```java
public String toString() {
// Create a copy, don't share the array
return new String(value, 0, count);
}
```

        它是**通过new String\(\)的方式**来作为值进行返回的，所以是在堆中开辟的一块空间。所以和常量池中的不一样。结果是false。

**特例1：**

```java
public static final String s1="abc";
public static final String s2="def";
public static void main(String[] args) {
String s3=s1+s2;
String s4="abcdef";
System.out.println(s3==s4);          //true
}
```

  因为s1和s2都是final类型的且在编译阶段都是已经复制了，所以相当于一个常量。

**特例2：**

```java
public static final String s1;
public static final String s2;
static{
s1="abc";
s2="def";
}
public static void main(String[] args) {
String s3=s1+s2;
String s4="abcdef";
System.out.println(s3==s4);            //false
}
```

        虽然s1和s2都是final类型的但是一开始没有初始化，在编译期还不可以知道具体的值，还是变量，所以什么时候赋值，赋什么值都是个变数。所以是false。

###  2. 使用new关键字

```java
String s1=new String("abc");
String s2=new String("abc");
System.out.println(s1==s2);             //false
```

对于，

```java
String s1=new String("abc");
```

 **这条语句创建了两个字符串对象**：

1. 一个是“abc”这个**直接量对应的字符串对象**： 类进行**加载时**，会把字符串abc放进全局的常量池中，进行保存。
2. 一个是由new String\(\)构造器返回的**字符串对象**： **运行时**，当你运行程序的时候，常量池中存在字符串abc,于是把字面量abc拿进heap中，使它的引用s1。

### 3. 动态添加

      String类的intern\(\)方法：

         如果常量池中存在这个对象直接返回该对象的引用，如果没有添加，再返回该对象的引用：

```java
String s1=new String("abc");
String s2=s1.intern();
String s3="abc";
System.out.println(s2==s3);     //true
```

        


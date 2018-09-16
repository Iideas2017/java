# 字符串常量池

## **1、字符串池**

　　字符串的分配，和其他的对象分配一样，耗费高昂的时间与空间代价。JVM为了提高性能和减少内存开销，在实例化**字符串字面值**的时候进行了一些优化。为了减少在JVM中创建的字符串的数量，字符串类维护了一个字符串常量池，每当**以字面值形式**创建一个字符串时，JVM会首先检查字符串常量池：如果字符串已经存在池中，就返回池中的实例引用；如果字符串不在池中，就会实例化一个字符串并放到池中。

**Java能够进行这样的优化是因为字符串是不可变的，可以不用担心数据冲突进行共享。** 例如：

```java
public class Program
{
    public static void main(String[] args)
    {
       String str1 = "Hello";  
       String str2 = "Hello"; 
       System.out.print(str1 == str2);   // true
    }
}
```

　　一个初始为空的字符串池，它由类 String 私有地维护。当以字面值形式创建一个字符串时，总是先检查字符串池是否含存在该对象，若存在，则直接返回。此外，**通过 new 操作符创建的字符串对象不指向字符串池中的任何对象**。

## **2、手动入池**

　　**一个初始为空的字符串池，它由类 String 私有地维护。** 当调用 intern 方法时，如果池已经包含一个等于此 String 对象的字符串（用 equals\(Object\) 方法确定），则返回池中的字符串。否则，将此 String 对象添加到池中，并返回此 String 对象的引用。

特别地，手动入池遵循以下规则：

　　**对于任意两个字符串 s 和 t ，当且仅当 s.equals\(t\) 为 true 时，s.intern\(\) == t.intern\(\) 才为 true 。** 　　

```java
public class TestString{
    public static void main(String args[]){
        String str1 = "abc";
        String str2 = new String("abc");
        String str3 = s2.intern();

        System.out.println( str1 == str2 );   //false
        System.out.println( str1 == str3 );   //true
    }
}
```

　　所以，对于 String str1 = “abc”，str1 引用的是 **常量池（方法区）** 的对象；而 String str2 = new String\(“abc”\)，str2引用的是 **堆** 中的对象，所以内存地址不一样。但是由于内容一样，所以 str1 和 str3 指向同一对象。

## **3、实例**

　　看下面几个场景来深入理解 String。

### 1\) 情景一：字符串常量池

　　Java虚拟机\(JVM\)中存在着一个字符串常量池，其中保存着很多String对象，并且这些String对象可以被共享使用，因此提高了效率。之所以字符串具有字符串常量池，是因为String对象是不可变的，因此可以被共享。字符串常量池由String类维护，我们可以通过intern\(\)方法使字符串池手动入池。

```java
    String s1 = "abc";     
    //↑ 在字符串池创建了一个对象  
    String s2 = "abc";     
    //↑ 字符串pool已经存在对象“abc”(共享),所以创建0个对象，累计创建一个对象  
    System.out.println("s1 == s2 : "+(s1==s2));    
    //↑ true 指向同一个对象，  
    System.out.println("s1.equals(s2) : " + (s1.equals(s2)));    
    //↑ true  值相等  12345678
```

### 2\) 情景二：关于new String\(“…”\)

```java
    String s3 = new String("abc");  
    //↑ 创建了两个对象，一个存放在字符串池中，一个存在与堆区中；  
    //↑ 还有一个对象引用s3存放在栈中  
    String s4 = new String("abc");  
    //↑ 字符串池中已经存在“abc”对象，所以只在堆中创建了一个对象  
    System.out.println("s3 == s4 : "+(s3==s4));  
    //↑false   s3和s4栈区的地址不同，指向堆区的不同地址；  
    System.out.println("s3.equals(s4) : "+(s3.equals(s4)));  
    //↑true  s3和s4的值相同  
    System.out.println("s1 == s3 : "+(s1==s3));  
    //↑false 存放的地区都不同，一个方法区，一个堆区  
    System.out.println("s1.equals(s3) : "+(s1.equals(s3)));  
    //↑true  值相同 12345678910111213
```

　　通过上一篇博文我们知道，通过 new String\(“…”\) 来创建字符串时，在该构造函数的参数值为字符串字面值的前提下，若该字面值不在字符串常量池中，那么会创建两个对象：一个在字符串常量池中，一个在堆中；否则，只会在堆中创建一个对象。对于不在同一区域的两个对象，二者的内存地址必定不同。

### 3\) 情景三：字符串连接符“+”

```java
    String str2 = "ab";  //1个对象  
    String str3 = "cd";  //1个对象                                         
    String str4 = str2+str3;                                        
    String str5 = "abcd";    
    System.out.println("str4 = str5 : " + (str4==str5)); // false  12345
```

　　我们看这个例子，局部变量 str2，str3 指向字符串常量池中的两个对象。在运行时，**第三行代码\(str2+str3\)实质上会被分解成五个步骤，分别是：**

　\(1\). 调用 String 类的静态方法 **String.valueOf\(\)** 将 str2 转换为字符串表示；

　\(2\). JVM 在堆中创建一个 StringBuilder对象，同时用str2指向转换后的字符串对象进行初始化；　

　\(3\). 调用StringBuilder对象的append方法完成与str3所指向的字符串对象的合并；

　\(4\). 调用 StringBuilder 的 toString\(\) 方法在堆中创建一个 String对象；

　\(5\). 将刚刚生成的String对象的堆地址存赋给局部变量引用str4。

　　而引用str5指向的是字符串常量池中字面值”abcd”所对应的字符串对象。由上面的内容我们可以知道，引用str4和str5指向的对象的地址必定不一样。这时，内存中实际上会存在五个字符串对象： 三个在字符串常量池中的String对象、一个在堆中的String对象和一个在堆中的StringBuilder对象。

### 4\) 情景四：字符串的编译期优化

```java
    String str1 = "ab" + "cd";  //1个对象  
    String str11 = "abcd";   
    System.out.println("str1 = str11 : "+ (str1 == str11));   // true

    final String str8 = "cd";  
    String str9 = "ab" + str8;  
    String str89 = "abcd";  
    System.out.println("str9 = str89 : "+ (str9 == str89));     // true
    //↑str8为常量变量，编译期会被优化  

    String str6 = "b";  
    String str7 = "a" + str6;  
    String str67 = "ab";  
    System.out.println("str7 = str67 : "+ (str7 == str67));     // false
    //↑str6为变量，在运行期才会被解析。123456789101112131415
```

　　Java 编译器对于类似**“常量+字面值”的组合，其值在编译的时候就能够被确定了。**在这里，str1 和 str9 的值在编译时就可以被确定，因此它们分别等价于： String str1 = “abcd”; 和 String str9 = “abcd”;

　　**Java 编译器对于含有 “String引用”的组合**，则在运行期会产生新的对象 **\(通过调用StringBuilder类的toString\(\)方法\)**，因此这个对象存储在堆中。

## **4、小结**

* **使用字面值形式创建的字符串与通过 new 创建的字符串一定是不同的，因为二者的存储位置不同：前者在方法区，后者在堆；**
* 我们在使用诸如String str = “abc”；的格式创建字符串对象时，总是想当然地认为，我们创建了String类的对象str。但是事实上， **对象可能并没有被创建。唯一可以肯定的是，指向 String 对象 的引用被创建了。**至于这个引用到底是否指向了一个新的对象，必须根据上下文来考虑；
* **字符串常量池的理念是** [**《享元模式》**](http://blog.csdn.net/justloveyou_/article/details/55045638)**；**
* **Java 编译器对 “常量+字面值” 的组合 是当成常量表达式直接求值来优化的；对于含有“String引用”**的组合，其在编译期不能被确定，会在运行期创建新对象。


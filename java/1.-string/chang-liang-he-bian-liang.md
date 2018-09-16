# 常量和变量

## 1. 变量

     **内存地址不变,值可以改变的东西称为变量**，换句话说，在内存地址不变的前提下内存的内容是可变的，例如：

```java
public class String_2 {  
    public static void f(){  
        Human_1 h = new Human_1(1,30);  
        Human_1 h2 = h; 
        System.out.printf("h: %s\n", h.toString());   
        System.out.printf("h2: %s\n\n", h.toString());   

        h.id = 3;  
        h.age = 32;  
        System.out.printf("h: %s\n", h.toString());   
        System.out.printf("h2: %s\n\n", h.toString());   

        System.out.println( h == h2 );   
        // true : 引用值不变，即对象内存底子不变，但内容改变
    }
} 
```

## 2. 常量

       我们一般把**若内存地址不变, 则值也不可以改变的东西称为常量**，典型的 String 就是不可变的，所以称之为 **常量\(constant\)**。此外，**我们可以通过final关键字来定义常量，但严格来说，只有基本类型被其修饰后才是常量（对基本类型来说是其值不可变，而对于对象变量来说其引用不可再变）。** 

```java
final int i = 5;
```


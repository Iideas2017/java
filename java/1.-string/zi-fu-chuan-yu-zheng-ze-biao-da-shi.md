# 2.6 字符串与正则表达式

     **正则表达式用一个字符串来描述一个特征，然后去验证另一个字符串是否符合这个特征。**使用正则表达式，我们能够以编程的方式，构造复杂的文本模式，并对输入的字符串进行搜索。Java 内置了对正则表达式的支持，其相关的类库在 java.util.regex 包下，感兴趣的读者可以去查看相应的 API 和 JDK 源码。在使用过程中，有两点需要注意以下：

## 1、Java转义与正则表达式转义

　　要想匹配某些特殊字符，比如 “\”，需要进行两次转义，即Java转义与正则表达式转义。对于下面的例子，需要注意的是，split\(\)函数的参数必须是“正则表达式”字符串。

```java
public class Test {
    public static void main(String[] args) {
        String a = "a\\b";
        System.out.println(a.split("\\\\")[0]);
        System.out.println(a.split("\\\\")[1]);
    }
}/* Output: 
    a
    b
 *///
```

## 2、使用 Pattern 与 Matcher 构造功能强大的正则表达式对象

　　Pattern 与 Matcher的组合就是Java对正则表达式的主要内置支持，如下：

```java
public class Test {
    public static void main(String[] args) {
        Pattern pattern = Pattern.compile(".\\\\.");
        Matcher matcher = pattern.matcher("a\\b");

        System.out.println(matcher.matches());
        System.out.println(matcher.group());
    }
}/* Output: 
    true
    a\b
 *///:~
```


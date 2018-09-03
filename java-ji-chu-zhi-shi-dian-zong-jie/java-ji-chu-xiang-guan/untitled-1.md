# 数据类型

Java中的数据类型分两种：基本数据类型和引用数据类型。

## 1. **基本数据类型** 

对于这些都是用的_==_来比较两者的值是不是相等。（byte short int long char float double boolean）

## 2.  引用数据类型

           一般情况下，equals和==是一样的都是比较的两者的地址值是不是一样。但是有特殊的情况：我们都知道我们使用的类都是继承自Object基类，Object中equals方法中是使用==来实现的，即比较的是两者的地址值。但是Object的子类可以重写equals方法，比如Date、String、Integer等类都是重写了equals都是重写了，比较的是值是否相等。例如String类的源码：

{% page-ref page="../bu-fen-yuan-ma/string-yuan-ma-xue-xi.md" %}


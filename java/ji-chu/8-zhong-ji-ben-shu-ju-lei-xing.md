# 8种基本数据类型

### 1.使用了常量池技术的： 

        Byte,Short,Integer,Long,Character,Boolean都实现了常量池技术

#### 1&gt;不使用new关键字

```java
private static class IntegerCache {
static final int high;
static final Integer cache[];
static {
final int low = -128;
// high value may be configured by property
int h = 127;
if (integerCacheHighPropValue != null) {
// Use Long.decode here to avoid invoking methods that
// require Integer's autoboxing cache to be initialized
int i = Long.decode(integerCacheHighPropValue).intValue();
i = Math.max(i, 127);
// Maximum array size is Integer.MAX_VALUE
h = Math.min(i, Integer.MAX_VALUE - -low);
}
high = h;
cache = new Integer[(high - low) + 1];
int j = low;
for(int k = 0; k < cache.length; k++)
cache[k] = new Integer(j++);
}
private IntegerCache() {}
}
```

 进行了缓存，范围是\[-128,127\],只要是这个范围内的数字都会缓存到这个里面，做成常量池进行管理。

```java
Integer i1=10;
Integer i2=10;
System.out.println(i1==i2);                 //true

int i1=10;
Integer i2=10;// 1.自动装箱
System.out.println(i1==i2);//2.自动拆箱      //true（在-128到127 范围内的）
```

自动装箱：

```java
public static Integer valueOf(int i) {
final int offset = 128;
if (i >= -128 && i <= 127) {               // must cache（注意范围）
return IntegerCache.cache[i + offset];
}
return new Integer(i);
}
```

####  2&gt;.使用new关键字：

        如果使用了new关键字就是在堆内存中开辟了一块内存。每次new一个都是在堆中开辟一块内存， 所以每一个的地址都不一样。

```java
Integer i1=new Integer(10);
Integer i2=new Integer(10);
System.out.println(i1==i2);                //false
```

### 2.  没有实现常量池的Float和Double

```java
Float f1=10.0f;
Float f2=10.0f; 
System.out.println(f1==f2);                //false
Double d1=12.0;
Double d2=12.0;
System.out.println(d1==d2);                //false
```




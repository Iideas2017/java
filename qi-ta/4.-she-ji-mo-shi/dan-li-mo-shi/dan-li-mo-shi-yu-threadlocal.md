# 单例模式与ThreadLocal

　　借助于 ThreadLocal，我们可以实现**双重检查模式**的变体。我们将**临界资源instance线程私有化\(局部化\)**，具体到本例就是将双重检测的**第一层检测条件 if \(instance == null\) 转换为线程局部范围内的操作**，对应的代码清单如下：

```java
public class Singleton {

    // ThreadLocal 线程局部变量，将单例instance线程私有化
    private static ThreadLocal<Singleton> threadlocal = new ThreadLocal<Singleton>();
    private static Singleton instance;

    private Singleton() {

    }

    public static Singleton getInstance() {

        // 第一次检查：若线程第一次访问，则进入if语句块；否则，若线程已经访问过，则直接返回ThreadLocal中的值
        if (threadlocal.get() == null) {
            synchronized (Singleton.class) {
                if (instance == null) {  // 第二次检查：该单例是否被创建
                    instance = new Singleton();
                }
            }
            threadlocal.set(instance); // 将单例放入ThreadLocal中
        }
        return threadlocal.get();
    }
}/* Output(完全一致): 
        1028355155
        1028355155
        1028355155
        1028355155
        1028355155
        1028355155
        1028355155
        1028355155
        1028355155
        1028355155
*///
```

　　借助于 ThreadLocal，我们也可以实现线程安全的懒汉式单例。但与直接双重检查模式使用，本实现在效率上还不如后者。


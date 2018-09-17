# 多线程环境下单例模式

　　在单线程环境下，无论是饿汉式单例还是懒汉式单例，它们都能够正常工作。但是，在多线程环境下，情形就发生了变化：

       由于**饿汉式单例天生就是线程安全**的，可以直接用于多线程而不会出现问题；

       但懒汉式单例本身是非线程安全的，因此就会出现多个实例的情况，与单例模式的初衷是相背离的。

　　特别地，为了能够更好的观察到单例模式的实现是否是线程安全的，我们提供了一个简单的测试程序来验证。该示例程序的判断原理是：

　　开启多个线程来分别获取单例，然后打印它们所获取到的单例的hashCode值。若它们获取的单例是相同的\(该单例模式的实现是线程安全的\)，那么它们的hashCode值一定完全一致；若它们的hashCode值不完全一致，那么获取的单例必定不是同一个，即该单例模式的实现不是线程安全的，是多例的。注意，相应输出结果附在每个单例模式实现示例后。 

```java
public class Test {
    public static void main(String[] args) {
        Thread[] threads = new Thread[10];
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new TestThread();
        }

        for (int i = 0; i < threads.length; i++) {
            threads[i].start();
        }
    }

}

class TestThread extends Thread {
    @Override
    public void run() {
        // 对于不同单例模式的实现，只需更改相应的单例类名及其公有静态工厂方法名即可
        int hash = Singleton5.getSingleton5().hashCode();  
        System.out.println(hash);
    }
}
```

## **1、饿汉式单例天生线程安全**

```java
// 饿汉式单例
public class Singleton1 {

    // 指向自己实例的私有静态引用，主动创建
    private static Singleton1 singleton1 = new Singleton1();

    // 私有的构造方法
    private Singleton1(){}

    // 以自己实例为返回值的静态的公有方法，静态工厂方法
    public static Singleton1 getSingleton1(){
        return singleton1;
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

　　我们已经在上面提到，类加载的方式是按需加载，且只加载一次。因此，在上述单例类被加载时，就会实例化一个对象并交给自己的引用，供系统使用。换句话说，**在线程访问单例对象之前就已经创建好了。**再加上，由于一个类在整个生命周期中只会被加载一次，因此该单例类只会创建一个实例，也就是说，线程每次都只能也必定只可以拿到这个唯一的对象。**因此就说，饿汉式单例天生就是线程安全的**。

## **2、懒汉式单例是非线程安全的**

```java
// 传统懒汉式单例
public class Singleton2 {

    // 指向自己实例的私有静态引用
    private static Singleton2 singleton2;

    // 私有的构造方法
    private Singleton2(){}

    // 以自己实例为返回值的静态的公有方法，静态工厂方法
    public static Singleton2 getSingleton2(){
        // 被动创建，在真正需要使用时才去创建
        if (singleton2 == null) {
            singleton2 = new Singleton2();
        }
        return singleton2;
    }
}/* Output(不完全一致): 
        1084284121
        2136955031
        2136955031
        1104499981
        298825033
        298825033
        2136955031
        482535999
        298825033
        2136955031
 *///
```

　　上面发生非线程安全的一个显著原因是，会有多个线程同时进入 if \(singleton2 == null\) {…} 语句块的情形发生。**当这种这种情形发生后，该单例类就会创建出多个实例，违背单例模式的初衷。因此，传统的懒汉式单例是非线程安全的。**

## **3、实现线程安全的懒汉式单例**

### 1\)、同步延迟加载 — synchronized方法

```java
// 线程安全的懒汉式单例
public class Singleton2 {

    private static Singleton2 singleton2;

    private Singleton2(){}

    // 使用 synchronized 修饰，临界资源的同步互斥访问
    public static synchronized Singleton2 getSingleton2(){
        if (singleton2 == null) {
            singleton2 = new Singleton2();
        }
        return singleton2;
    }
}/* Output(完全一致): 
        1104499981
        1104499981
        1104499981
        1104499981
        1104499981
        1104499981
        1104499981
        1104499981
        1104499981
        1104499981
 *///
```

　　该实现与上面传统懒汉式单例的实现唯一的差别就在于：是否使用 synchronized 修饰 getSingleton2\(\)方法。若使用，就保证了**对临界资源的同步互斥访问，也就保证了单例**。

　　从执行结果上来看，问题已经解决了，但是这种实现方式的运行效率会很低，因为同步块的作用域有点大，而且锁的粒度有点粗。同步方法效率低，那我们考虑使用同步代码块来实现。

### 2\)、同步延迟加载 — synchronized块

```java
// 线程安全的懒汉式单例
public class Singleton2 {

    private static Singleton2 singleton2;

    private Singleton2(){}


    public static Singleton2 getSingleton2(){
        synchronized(Singleton2.class){  
        // 使用 synchronized 块，临界资源的同步互斥访问
            if (singleton2 == null) { 
                singleton2 = new Singleton2();
            }
        }
        return singleton2;
    }
}/* Output(完全一致): 
        16993205
        16993205
        16993205
        16993205
        16993205
        16993205
        16993205
        16993205
        16993205
        16993205
 *///
```

　　该实现与上面synchronized方法版本实现类似，此不赘述。从执行结果上来看，问题已经解决了，但是这种实现方式的运行效率仍然比较低，事实上，和使用synchronized方法的版本相比，基本没有任何效率上的提高。

### 3\)、同步延迟加载 — 使用内部类实现延迟加载

```java
// 线程安全的懒汉式单例
public class Singleton5 {

    // 私有内部类，按需加载，用时加载，也就是延迟加载
    private static class Holder {
        private static Singleton5 singleton5 = new Singleton5();
    }

    private Singleton5() {

    }

    public static Singleton5 getSingleton5() {
        return Holder.singleton5;
    }
}
/* Output(完全一致): 
        482535999
        482535999
        482535999
        482535999
        482535999
        482535999
        482535999
        482535999
        482535999
        482535999
 *///
```

　　如上述代码所示，我们可以使用内部类实现线程安全的懒汉式单例，这种方式也是一种效率比较高的做法，它与**饿汉式单例**的区别就是：这种方式不但是线程安全的，还是**延迟加载的，真正做到了用时才初始化**。

　　当客户端调用getSingleton5\(\)方法时，会触发Holder类的初始化。由于singleton5是Hold的类成员变量，因此在JVM调用Holder类的类构造器对其进行初始化时，虚拟机会保证一个类的**类构造器**在多线程环境中被正确的加锁、同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的类构造器，其他线程都需要阻塞等待，直到活动线程执行方法完毕。在这种情形下，其他线程虽然会被阻塞，但如果执行类构造器方法的那条线程退出后，其他线程在唤醒之后不会再次进入/执行类构造器，因为 在同一个类加载器下，一个类型只会被初始化一次，因此就保证了单例。

## 


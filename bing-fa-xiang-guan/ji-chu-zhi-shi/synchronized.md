# Synchronized 关键字（上）

## 一. synchronized 同步方法或者同步块

　　在了解 synchronized 关键字的使用方法之前，我们先来看一个概念：**互斥锁**，即 **能到达到互斥访问目的的锁。**举个简单的例子，如果对临界资源加上互斥锁，当一个线程在访问该临界资源时，其他线程便只能等待。

　　在 Java 中，可以使用 synchronized 关键字来标记一个方法或者代码块，当某个线程调用该对象的synchronized方法或者访问synchronized代码块时，这个线程便获得了该对象的锁，其他线程暂时无法访问这个方法，只有等待这个方法执行完毕或者代码块执行完毕，这个线程才会释放该对象的锁，其他线程才能执行这个方法或者代码块。

### 1\) synchronized方法

　　不过需要注意以下三点：

　　1）**当一个线程正在访问一个对象的 synchronized 方法，那么其他线程不能访问该对象的其他 synchronized 方法。**这个原因很简单，因为一个对象只有一把锁，当一个线程获取了该对象的锁之后，其他线程无法获取该对象的锁，所以无法访问该对象的其他synchronized方法。

　　2）**当一个线程正在访问一个对象的 synchronized 方法，那么其他线程能访问该对象的非 synchronized 方法。**这个原因很简单，访问非 synchronized 方法不需要获得该对象的锁，假如一个方法没用 synchronized 关键字修饰，说明它不会使用到临界资源，那么其他线程是可以访问这个方法的，

　　3）**如果一个线程 A 需要访问对象 object1 的 synchronized 方法 fun1，另外一个线程 B 需要访问对象 object2 的 synchronized 方法 fun1，即使 object1 和 object2 是同一类型），也不会产生线程安全问题，因为他们访问的是不同的对象，所以不存在互斥问题。**

### 2\) synchronized 同步块

　　synchronized 代码块类似于以下这种形式：

```java
synchronized (lock){
    //访问共享可变资源
    ...
}
```

　　当在某个线程中执行这段代码块，该线程会获取对象lock的锁，从而使得其他线程无法同时访问该代码块。其中，lock 可以是 this，代表获取当前对象的锁，也可以是类中的一个属性，代表获取该属性的锁。特别地， **实例同步方法** 与 **synchronized\(this\)同步块** 是互斥的，因为它们锁的是同一个对象。但与 **synchronized\(非this\)同步块** 是异步的，因为它们锁的是不同对象。

　　从上面代码可以看出，**synchronized代码块 比 synchronized方法 的粒度更细一些，使用起来也灵活得多。**因为也许一个方法中只有一部分代码只需要同步，如果此时对整个方法用synchronized进行同步，会影响程序执行效率。而使用synchronized代码块就可以避免这个问题，synchronized代码块可以实现只对需要同步的地方进行同步。

### 3\) class 对象锁

　　**特别地，每个类也会有一个锁，静态的 synchronized方法 就是以Class对象作为锁。另外，它可以用来控制对 static 数据成员 （static 数据成员不专属于任何一个对象，是类成员）** 的并发访问。并且，如果一个线程执行一个对象的非static synchronized 方法，另外一个线程需要执行这个对象所属类的 static synchronized 方法，也不会发生互斥现象。因为访问 static synchronized 方法占用的是类锁，而访问非 static synchronized 方法占用的是对象锁，所以不存在互斥现象。例如，

```java
public class Test {

    public static void main(String[] args)  {
        final InsertData insertData = new InsertData();
        new Thread(){
            @Override
            public void run() {
                insertData.insert();
            }
        }.start(); 
        new Thread(){
            @Override
            public void run() {
                insertData.insert1();
            }
        }.start();
    }  
}

class InsertData { 

    // 非 static synchronized 方法
    public synchronized void insert(){
        System.out.println("执行insert");
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("执行insert完毕");
    }

    // static synchronized 方法
    public synchronized static void insert1() {
        System.out.println("执行insert1");
        System.out.println("执行insert1完毕");
    }
}/* Output: 
        执行insert
        执行insert1
        执行insert1完毕
        执行insert完毕
 *///
```

　　根据执行结果，我们可以看到第一个线程里面执行的是insert方法，不会导致第二个线程执行insert1方法发生阻塞现象。下面，我们看一下 synchronized 关键字到底做了什么事情，我们来反编译它的字节码看一下，下面这段代码反编译后的字节码为：

```java
public class InsertData {
    private Object object = new Object();

    public void insert(Thread thread){
        synchronized (object) {}
    }

    public synchronized void insert1(Thread thread){}

    public void insert2(Thread thread){}
}
```

![](../../.gitbook/assets/image%20%28165%29.png)



从反编译获得的字节码可以看出，**synchronized 代码块实际上多了 monitorenter 和 monitorexit 两条指令。** monitorenter指令执行时会让对象的锁计数加1，而monitorexit指令执行时会让对象的锁计数减1，其实这个与操作系统里面的PV操作很像，操作系统里面的PV操作就是用来控制多个进程对临界资源的访问。对于synchronized方法，执行中的线程识别该方法的 method\_info 结构是否有 ACC\_SYNCHRONIZED 标记设置，然后它自动获取对象的锁，调用方法，最后释放锁。如果有异常发生，线程自动释放锁。

　　有一点要注意：**对于 synchronized方法 或者 synchronized代码块，当出现异常时，JVM会自动释放当前线程占用的锁，因此不会由于异常导致出现死锁现象。**　　

## 二. 可重入性

　　一般地，当某个线程请求一个由其他线程持有的锁时，发出请求的线程就会阻塞。然而，由于 Java 的内置锁是可重入的，因此如果某个线程试图获得一个已经由它自己持有的锁时，那么这个请求就会成功。**可重入锁最大的作用是避免死锁。**

## 三. 注意事项

### **1\). 内置锁与字符串常量**

　　**由于字符串常量池的原因，在大多数情况下，同步synchronized代码块 都不使用 String 作为锁对象**，而改用其他，比如 new Object\(\) 实例化一个 Object 对象，因为它并不会被放入缓存中。看下面的例子：

```java
//资源类
class Service {
    public void print(String stringParam) {
        try {
            synchronized (stringParam) {
                while (true) {
                    System.out.println(Thread.currentThread().getName());
                    Thread.sleep(1000);
                }
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

//线程A
class ThreadA extends Thread {
    private Service service;

    public ThreadA(Service service) {
        super();
        this.service = service;
    }

    @Override
    public void run() {
        service.print("AA");
    }
}

//线程B
class ThreadB extends Thread {
    private Service service;

    public ThreadB(Service service) {
        super();
        this.service = service;
    }

    @Override
    public void run() {
        service.print("AA");
    }
}

//测试
public class Run {
    public static void main(String[] args) {

        //临界资源
        Service service = new Service();

        //创建并启动线程A
        ThreadA a = new ThreadA(service);
        a.setName("A");
        a.start();

        //创建并启动线程B
        ThreadB b = new ThreadB(service);
        b.setName("B");
        b.start();

    }
}/* Output (死锁): 
        A
        A
        A
        A
        ...
 *///
```

　　出现上述结果就是因为 String 类型的参数都是 “AA”，两个线程持有相同的锁，所以 线程B 始终得不到执行，造成死锁。进一步地，所谓死锁是指：   
　   **不同的线程都在等待根本不可能被释放的锁，从而导致所有的任务都无法继续完成。**

### **b\). 锁的是对象而非引用**

　　在将任何数据类型作为同步锁时，需要注意的是，是否有多个线程将同时去竞争该锁对象：   
　　1\).若它们将同时竞争同一把锁，则这些线程之间就是同步的；   
　　2\).否则，这些线程之间就是异步的。

看下面的例子：

```java
//资源类
class MyService {
    private String lock = "123";

    public void testMethod() {
        try {
            synchronized (lock) {
                System.out.println(Thread.currentThread().getName() + " begin "
                    + System.currentTimeMillis());
                lock = "456";
                Thread.sleep(2000);
                System.out.println(Thread.currentThread().getName() + "   end "
                        + System.currentTimeMillis());
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

//线程B
class ThreadB extends Thread {

    private MyService service;

    public ThreadB(MyService service) {
        super();
        this.service = service;
    }

    @Override
    public void run() {
        service.testMethod();
    }
}

//线程A
class ThreadA extends Thread {

    private MyService service;

    public ThreadA(MyService service) {
        super();
        this.service = service;
    }

    @Override
    public void run() {
        service.testMethod();
    }
}

//测试
public class Run1 {
    public static void main(String[] args) throws InterruptedException {

        //临界资源
        MyService service = new MyService();

        //线程A
        ThreadA a = new ThreadA(service);
        a.setName("A");

        //线程B
        ThreadB b = new ThreadB(service);
        b.setName("B");

        a.start();
        Thread.sleep(50);// 存在50毫秒
        b.start();
    }
}/* Output(循环): 
       A begin 1484319778766
       B begin 1484319778815
       A   end 1484319780766
       B   end 1484319780815
 *///:~1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374757677
```

　　由上述结果可知，线程 A、B 是异步的。因为50毫秒过后， 线程B 取得的锁对象是 “456”，而 线程A 依然持有的锁对象是 “123”。所以，这两个线程是异步的。若将上述语句 “Thread.sleep\(50\);” 注释，则有：

```java
//测试
public class Run1 {
    public static void main(String[] args) throws InterruptedException {

        //临界资源
        MyService service = new MyService();

        //线程A
        ThreadA a = new ThreadA(service);
        a.setName("A");

        //线程B
        ThreadB b = new ThreadB(service);
        b.setName("B");

        a.start();
        // Thread.sleep(50);// 存在50毫秒
        b.start();
    }
}/* Output(循环): 
       B begin 1484319952017
       B   end 1484319954018
       A begin 1484319954018
       A   end 1484319956019
 *///:~12345678910111213141516171819202122232425
```

　　由上述结果可知，线程 A、B 是同步的。因为线程 A、B 竞争的是同一个锁“123”，虽然先获得运行的线程将 lock 指向了 对象“456”，但结果还是同步的。因为线程 A 和 B 共同争抢的锁对象是“123”，也就是说，**锁的是对象而非引用。**

## 四. 总结

　　用一句话来说，**synchronized 内置锁** 是一种 **对象锁** \(锁的是对象而非引用\)， **作用粒度是对象** ，可以用来实现对 **临界资源的同步互斥访问** ，是 **可重入** 的。特别地，对于 **临界资源** 有：

* **若该资源是静态的，即被 static 关键字修饰，那么访问它的方法必须是同步且是静态的，synchronized 块必须是 class锁；**
* **若该资源是非静态的，即没有被 static 关键字修饰，那么访问它的方法必须是同步的，synchronized 块是实例对象锁；**  　　

**实质上，关键字synchronized 主要包含两个特征：**

* **互斥性：**保证在同一时刻，只有一个线程可以执行某一个方法或某一个代码块；
* **可见性：**保证线程工作内存中的变量与公共内存中的变量同步，使多线程读取共享变量时可以获得最新值的使用。


# 线程安全问题

## 一. 线程安全问题

　　在单线程中不会出现线程安全问题，而在多线程编程中，有可能会出现同时访问同一个 **共享、可变资源** 的情况，这种资源可以是：一个变量、一个对象、一个文件等。特别注意两点，

* **共享：** 意味着该资源可以由多个线程同时访问；
* **可变：** 意味着该资源可以在其生命周期内被修改。

  　所以，当多个线程同时访问这种资源的时候，就会存在一个问题：

  　　　**由于每个线程执行的过程是不可控的，所以需要采用同步机制来协同对对象可变状态的访问。**   
　　

举个 **数据脏读** 的例子：

```java
//资源类
class PublicVar {

    public String username = "A";
    public String password = "AA";

    //同步实例方法
    public synchronized void setValue(String username, String password) {
        try {
            this.username = username;
            Thread.sleep(5000);
            this.password = password;

            System.out.println("method=setValue " +"\t" + "threadName="
                    + Thread.currentThread().getName() + "\t" + "username="
                    + username + ", password=" + password);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    //非同步实例方法
    public void getValue() {
        System.out.println("method=getValue " + "\t" +  "threadName="
                + Thread.currentThread().getName()+ "\t" + " username=" + username
                + ", password=" + password);
    }
}


//线程类
class ThreadA extends Thread {

    private PublicVar publicVar;

    public ThreadA(PublicVar publicVar) {
        super();
        this.publicVar = publicVar;
    }

    @Override
    public void run() {
        super.run();
        publicVar.setValue("B", "BB");
    }
}


//测试类
public class Test {

    public static void main(String[] args) {
        try {
            //临界资源
            PublicVar publicVarRef = new PublicVar();

            //创建并启动线程
            ThreadA thread = new ThreadA(publicVarRef);
            thread.start();

            Thread.sleep(200);// 打印结果受此值大小影响

            //在主线程中调用
            publicVarRef.getValue();

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}/* Output ( 数据交叉 ): 
        method=getValue     threadName=main         username=B, password=AA
        method=setValue     threadName=Thread-0     username=B, password=BB
 *///:
```

　　由程序输出可知，虽然在写操作进行了同步，但在读操作上仍然有可能出现一些意想不到的情况，例如上面所示的 **脏读**。发生 脏读 的情况是在执行读操作时，相应的数据已被其他线程 **部分修改** 过，导致 **数据交叉** 的现象产生。

　　这其实就是一个**线程安全问题**，即多个线程同时访问一个资源时，会导致程序运行结果并不是想看到的结果。这里面，这个资源被称为：**临界资源**。也就是说，当多个线程同时访问临界资源（一个对象，对象中的属性，一个文件，一个数据库等）时，就可能会产生线程安全问题。

　　不过，**当多个线程执行一个方法时，该方法内部的局部变量并不是临界资源，因为这些局部变量是在每个线程的私有栈中，因此不具有共享性，不会导致线程安全问题。**

## 二. 如何解决线程安全问题

　　实际上，所有的并发模式在解决线程安全问题时，采用的方案都是 **序列化访问临界资源** 。即在同一时刻，只能有一个线程访问临界资源，也称作 **同步互斥访问**。换句话说，就是在访问临界资源的代码前面加上一个锁，当访问完临界资源后释放锁，让其他线程继续访问。

　　**在 Java 中，提供了两种方式来实现同步互斥访问：synchronized 和 Lock。**

###  {#三-synchronized-同步方法或者同步块}


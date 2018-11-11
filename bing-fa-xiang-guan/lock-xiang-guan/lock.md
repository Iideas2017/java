# 2.1 Lock

         我们已经知道，synchronized 是java的关键字，是Java的内置特性，在JVM层面实现了对临界资源的同步互斥访问，但 synchronized **粒度有些大**，在处理实际问题时存在诸多局限性，比如响应中断等。Lock 提供了比 synchronized更广泛的锁操作，它能以更优雅的方式处理线程同步问题。

## 1. concurrent包的结构层次

其中包含了两个子包：atomic以及lock，另外在concurrent下的阻塞队列以及executors

![concurrent&#x76EE;&#x5F55;&#x7ED3;&#x6784;](../../.gitbook/assets/image%20%28294%29.png)

![concurrent&#x5305;&#x5B9E;&#x73B0;&#x6574;&#x4F53;&#x793A;&#x610F;&#x56FE;](../../.gitbook/assets/image%20%2875%29.png)

## 2. lock简介

```java
void lock();                                       //获取锁
void lockInterruptibly() throws InterruptedException；//获取锁的过程能够响应中断
boolean tryLock();//非阻塞式响应中断能立即返回，获取锁放回true反之返回fasle
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;//超时获取锁，在超时内或者未中断的情况下能够获取锁
void unlock();
Condition newCondition();//获取与lock绑定的等待通知组件，当前线程必须获得了锁才能进行等待，进行等待时会先释放锁，当再次获取锁时才能从等待中返回
```

### （1\) lock\(\)

　　首先，lock\(\)方法是平常使用得最多的一个方法，就是用来获取锁。如果锁已被其他线程获取，则进行等待。在前面已经讲到，**如果采用Lock，必须主动去释放锁，并且在发生异常时，不会自动释放锁。**因此，一般来说，**使用Lock必须在try…catch…块中进行，并且将释放锁的操作放在finally块中进行，以保证锁一定被被释放，防止死锁的发生。**

通常使用Lock来进行同步的话，是以下面这种形式去使用的：

```java
Lock lock = ...;
lock.lock();
try{
    //处理任务
}catch(Exception ex){

}finally{
    lock.unlock();   //释放锁
}
```

### （2\) tryLock\(\) & tryLock\(long time, TimeUnit unit\)

　　tryLock\(\)方法是有返回值的，它表示用来尝试获取锁，如果获取成功，则返回true；如果获取失败（即锁已被其他线程获取），则返回false，也就是说，**这个方法无论如何都会立即返回（在拿不到锁时不会一直在那等待）。**

　　tryLock\(long time, TimeUnit unit\)方法和tryLock\(\)方法是类似的，只不过区别在于**这个方法在拿不到锁时会等待一定的时间，在时间期限之内如果还拿不到锁，就返回false，同时可以响应中断。**如果一开始拿到锁或者在等待期间内拿到了锁，则返回true。

　　一般情况下，通过tryLock来获取锁时是这样使用的：

```java
Lock lock = ...;
if(lock.tryLock()) {
     try{
         //处理任务
     }catch(Exception ex){

     }finally{
         lock.unlock();   //释放锁
     } 
}else {
    //如果不能获取锁，则直接做其他事情
}
```

### （3\) lockInterruptibly\(\)

　　lockInterruptibly\(\)方法比较特殊，**当通过这个方法去获取锁时，如果线程正在等待获取锁，则这个线程能够响应中断，即中断线程的等待状态。**例如，当两个线程同时通过lock.lockInterruptibly\(\)想获取某个锁时，假若此时线程A获取到了锁，而线程B只有在等待，那么对线程B调用threadB.interrupt\(\)方法能够中断线程B的等待过程。

　　由于lockInterruptibly\(\)的声明中抛出了异常，所以lock.lockInterruptibly\(\)必须放在try块中或者在调用lockInterruptibly\(\)的方法外声明抛出 InterruptedException，但推荐使用后者，原因稍后阐述。因此，lockInterruptibly\(\)一般的使用形式如下：

```java
public void method() throws InterruptedException {
    lock.lockInterruptibly();
    try {  
     //.....
    }
    finally {
        lock.unlock();
    }  
}
```

　　注意，**当一个线程获取了锁之后，是不会被interrupt\(\)方法中断的。**因为interrupt\(\)方法只能中断阻塞过程中的线程而不能中断正在运行过程中的线程。因此，当通过lockInterruptibly\(\)方法获取某个锁时，如果不能获取到，那么只有进行等待的情况下，才可以响应中断的。与 synchronized 相比，当一个线程处于等待某个锁的状态，是无法被中断的，只有一直等待下去。

　　**最佳实践 \(Best Practice\)：**在使用Lock时，无论以哪种方式获取锁，习惯上最好一律将获取锁的代码放到 try…catch…，因为我们一般将锁的unlock操作放到finally子句中，如果线程没有获取到锁，在执行finally子句时，就会执行unlock操作，从而抛出 IllegalMonitorStateException，因为该线程并未获得到锁却执行了解锁操作。  



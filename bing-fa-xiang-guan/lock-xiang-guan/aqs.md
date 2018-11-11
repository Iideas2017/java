# 2.3 AQS

## 1. AQS工作原理概要

       内部通过一个int类型的成员变量**state**来控制同步状态,当state=0时，则说明没有任何线程占有共享资源的锁，当state=1时，则说明有线程目前正在使用共享变量，其他线程必须加入同步队列进行等待，AQS内部通过内部类Node构成FIFO的同步队列来完成线程获取锁的排队工作;

同时利用内部类ConditionObject构建等待队列，当Condition调用wait\(\)方法后，线程将会加入等待队列中，

而当Condition调用signal\(\)方法后，线程将从等待队列转移动同步队列中进行锁竞争。

注意这里涉及到两种队列，一种的**同步队列**，当线程**请求锁而等待的**后将加入同步队列等待，而另一种则是**等待队列\(**可有多个\)，通过Condition调用await\(\)方法释放锁后，将加入等待队列。

## 2. AQS的模板方法设计模式

       它的**子类必须重写AQS的几个protected修饰的用来改变同步状态的方法**，其他方法主要是实现了排队和阻塞机制。

       **状态的更新使用getState, setState 以及compareAndSetState 这三个方法**。

![AQS&#x53EF;&#x91CD;&#x5199;&#x7684;&#x65B9;&#x6CD5;](../../.gitbook/assets/image%20%28270%29.png)

AQS提供的模板方法可以分为3类：

1. 独占式获取与释放同步状态；
2. 共享式获取与释放同步状态；
3. 查询同步队列中等待线程情况；

同步组件通过AQS提供的模板方法实现自己的同步语义。

![AQS&#x63D0;&#x4F9B;&#x7684;&#x6A21;&#x677F;&#x65B9;&#x6CD5;](../../.gitbook/assets/image%20%28143%29.png)

它将**一些方法开放给子类进行重写，而同步器给同步组件所提供模板方法又会重新调用被子类所重写的方法**。

举个例子，AQS中需要重写的方法tryAcquire：

```java
protected boolean tryAcquire(int arg) {              //原tryAcquire
        throw new UnsupportedOperationException();
}
```

ReentrantLock中NonfairSync（继承AQS）会重写该方法为：

```java
protected final boolean tryAcquire(int acquires) {   //重写后tryAcquire
    return nonfairTryAcquire(acquires);
}
```

而AQS中的模板方法acquire\(\):

```java
 public final void acquire(int arg) {       
        if (!tryAcquire(arg) &&                 //重新调用被子类所重写的方法
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
 }
```

会调用tryAcquire方法，而此时当继承AQS的NonfairSync调用模板方法acquire时就会调用已经被NonfairSync重写的tryAcquire方法。这就是使用AQS的方式，在弄懂这点后会lock的实现理解有很大的提升。可以归纳总结为这么几点：

1. 同步组件（这里不仅仅值锁，还包括CountDownLatch等）的实现依赖于同步器AQS，在同步组件实现中，使用AQS的方式被推荐定义继承AQS的静态内存类；
2. AQS采用模板方法进行设计，AQS的protected修饰的方法需要由继承AQS的子类进行重写实现，当调用AQS的子类的方法时就会调用被重写的方法；
3. AQS负责同步状态的管理，线程的排队，等待和唤醒这些底层操作，而Lock等同步组件主要专注于实现同步语义；
4. 在重写AQS的方式时，使用AQS提供的`getState(),setState(),compareAndSetState()`方法进行修改同步状态。

## 3. 同步队列

 获取锁失败进行入队操作，获取锁成功进行出队操作

![&#x540C;&#x6B65;&#x961F;&#x5217;&#x6A21;&#x578B;](../../.gitbook/assets/image%20%28139%29.png)

* 其中head指向同步队列的头部，注意**head为空结点**，不存储信息。而tail则是同步队列的队尾，同步队列采用的是**双向链表**的结构这样可方便队列**进行结点增删操作**。
* **state**变量则是代表**同步**状态，执行当线程调用lock方法进行加锁后，如果此时state的值为0，则说明当前线程可以获取到锁\(在本篇文章中，锁和同步状态代表同一个意思\)，同时将state设置为1，表示获取成功。如果state已为1，也就是当前锁已被其他线程持有，那么当前执行线程将被封装为Node结点加入同步队列等待。
* 其中**Node结点**是对每一个访问同步代码的线程的封装，从图中的Node的数据结构也可看出，其包含了需要同步的线程本身以及线程的状态，如是否被阻塞，是否等待唤醒，是否已经被取消等。每个Node结点内部关联其前继结点prev和后继结点next，这样可以方便线程释放锁后快速唤醒下一个在等待的线程，**Node是AQS的内部类**。

### 3.1  独占式锁的获取

![](../../.gitbook/assets/image%20%28226%29.png)

![](../../.gitbook/assets/image%20%28161%29.png)

### 3.2 独占锁的释放

 首先获取头节点的后继节点，当后继节点的时候会调用LookSupport.unpark\(\)方法，该方法会唤醒该节点的后继节点所包装的线程。因此，**每一次锁释放后就会唤醒队列中该节点的后继节点所引用的线程，从而进一步可以佐证获得锁的过程是一个FIFO（先进先出）的过程。**

### **3.3 总结**

1. **线程获取锁失败，线程被封装成Node进行入队操作，核心方法在于addWaiter\(\)和enq\(\)，同时enq\(\)完成对同步队列的头结点初始化工作以及CAS操作失败的重试**;
2. **线程获取锁是一个自旋的过程，当且仅当 当前节点的前驱节点是头结点并且成功获得同步状态时，节点出队即该节点引用的线程获得锁，否则，当不满足条件时就会调用LookSupport.park\(\)方法使得线程阻塞**；
3. **释放锁的时候会唤醒后继节点。**

 **在获取同步状态时，AQS维护一个同步队列，获取同步状态失败的线程会加入到队列中进行自旋；移除队列（或停止自旋）的条件是前驱节点是头结点并且成功获得了同步状态。在释放同步状态时，同步器会调用unparkSuccessor\(\)方法唤醒后继节点。**

## 4. 等待队列

![&#x7B49;&#x5F85;&#x961F;&#x5217;](../../.gitbook/assets/image%20%2817%29.png)

**等待队列是一个单向队列;**

 **不带头结点的链式队列;**

 **对象Object对象监视器上只能拥有一个同步队列和一个等待队列，而并发包中的Lock拥有一个同步队列和多个等待队列**。

 当当前线程调用condition.await\(\)方法后，会使得当前线程释放lock然后加入到等待队列中，直至被signal/signalAll后会使得当前线程从等待队列中移至到同步队列中去，直到获得了lock后才会从await方法返回，或者在等待时被中断会做中断处理。

## 5. **自定义同步组件的静态内部类**

           同步器是用来构建锁和其他同步组件的基础框架，它的实现主要依赖一个int成员变量来表示同步状态以及通过一个FIFO队列构成等待队列 。

            它的**子类必须重写AQS的几个protected修饰的用来改变同步状态的方法**，其他方法主要是实现了排队和阻塞机制。**状态的更新使用getState,setState以及compareAndSetState这三个方法**。

          子类被**推荐定义为自定义同步组件的静态内部类**，同步器自身没有实现任何同步接口，它仅仅是**定义了若干同步状态的获取和释放方法**来供**自定义同步组件的使用**，同步器既支持独占式获取同步状态，也可以支持共享式获取同步状态，这样就可以方便的实现不同类型的同步组件。

          同步器是**实现锁**（也可以是任意同步组件）**的关键**，在锁的实现中聚合同步器，利用同步器实现锁的语义。可以这样理解二者的关系：**锁是面向使用者，它定义了使用者与锁交互的接口，隐藏了实现细节；同步器是面向锁的实现者，它简化了锁的实现方式，屏蔽了同步状态的管理，线程的排队，等待和唤醒等底层操作**。锁和同步器很好的隔离了使用者和实现者所需关注的领域。

## 6. 一个例子

下面使用一个例子来进一步理解下AQS的使用。这个例子也是来源于AQS源码中的example。

```java
class Mutex implements Lock, java.io.Serializable {
    // Our internal helper class
    // 继承AQS的静态内存类
    // 重写方法
    private static class Sync extends AbstractQueuedSynchronizer {
        // Reports whether in locked state
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        // Acquires the lock if state is zero
        public boolean tryAcquire(int acquires) {
            assert acquires == 1; // Otherwise unused
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        // Releases the lock by setting state to zero
        protected boolean tryRelease(int releases) {
            assert releases == 1; // Otherwise unused
            if (getState() == 0) throw new IllegalMonitorStateException();
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        // Provides a Condition
        Condition newCondition() {
            return new ConditionObject();
        }

        // Deserializes properly
        private void readObject(ObjectInputStream s)
                throws IOException, ClassNotFoundException {
            s.defaultReadObject();
            setState(0); // reset to unlocked state
        }
    }

    // The sync object does all the hard work. We just forward to it.
    private final Sync sync = new Sync();
    //使用同步器的模板方法实现自己的同步语义
    public void lock() {
        sync.acquire(1);
    }

    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    public void unlock() {
        sync.release(1);
    }

    public Condition newCondition() {
        return sync.newCondition();
    }

    public boolean isLocked() {
        return sync.isHeldExclusively();
    }

    public boolean hasQueuedThreads() {
        return sync.hasQueuedThreads();
    }

    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    public boolean tryLock(long timeout, TimeUnit unit)
            throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(timeout));
    }
}
```

MutexDemo：

```java
public class MutextDemo {
    private static Mutex mutex = new Mutex();

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            Thread thread = new Thread(() -> {
                mutex.lock();
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    mutex.unlock();
                }
            });
            thread.start();
        }
    }
}
```

执行情况：

![mutex&#x7684;&#x6267;&#x884C;&#x60C5;&#x51B5;](../../.gitbook/assets/image%20%28430%29.png)

上面的这个例子实现了独占锁的语义，在同一个时刻只允许一个线程占有锁。MutexDemo新建了10个线程，分别睡眠3s。从执行情况也可以看出来当前Thread-6正在执行占有锁而其他Thread-7,Thread-8等线程处于WAIT状态。按照推荐的方式，Mutex定义了一个**继承AQS的静态内部类Sync**,并且重写了AQS的tryAcquire等等方法，而对state的更新也是利用了setState\(\),getState\(\)，compareAndSetState\(\)这三个方法。在实现实现lock接口中的方法也只是调用了AQS提供的模板方法（因为Sync继承AQS）。从这个例子就可以很清楚的看出来，在同步组件的实现上主要是利用了AQS，而AQS“屏蔽”了同步状态的修改，线程排队等底层实现，通过AQS的模板方法可以很方便的给同步组件的实现者进行调用。而针对用户来说，只需要调用同步组件提供的方法来实现并发编程即可。同时在新建一个同步组件时需要把握的两个关键点是：

1. 实现同步组件时推荐定义继承AQS的静态内存类，并重写需要的protected修饰的方法；
2. 同步组件语义的实现依赖于AQS的模板方法，而AQS模板方法又依赖于被AQS的子类所重写的方法。

通俗点说，因为AQS整体设计思路采用模板方法设计模式，同步组件以及AQS的功能实际上别切分成各自的两部分：

**同步组件实现者的角度：**

通过可重写的方法：**独占式**： tryAcquire\(\)\(独占式获取同步状态），tryRelease\(\)（独占式释放同步状态）；**共享式** ：tryAcquireShared\(\)\(共享式获取同步状态\)，tryReleaseShared\(\)\(共享式释放同步状态\)；**告诉AQS怎样判断当前同步状态是否成功获取或者是否成功释放**。同步组件专注于对当前同步状态的逻辑判断，从而实现自己的同步语义。这句话比较抽象，举例来说，上面的Mutex例子中通过tryAcquire方法实现自己的同步语义，在该方法中如果当前同步状态为0（即该同步组件没被任何线程获取），当前线程可以获取同时将状态更改为1返回true，否则，该组件已经被线程占用返回false。很显然，该同步组件只能在同一时刻被线程占用，Mutex专注于获取释放的逻辑来实现自己想要表达的同步语义。

**AQS的角度**

而对AQS来说，只需要同步组件返回的true和false即可，因为AQS会对true和false会有不同的操作，true会认为当前线程获取同步组件成功直接返回，而false的话就AQS也会将当前线程插入同步队列等一系列的方法。

总的来说，同步组件通过重写AQS的方法实现自己想要表达的同步语义，而AQS只需要同步组件表达的true和false即可，AQS会针对true和false不同的情况做不同的处理，至于底层实现，可以[看这篇文章](https://www.jianshu.com/p/cc308d82cc71)。


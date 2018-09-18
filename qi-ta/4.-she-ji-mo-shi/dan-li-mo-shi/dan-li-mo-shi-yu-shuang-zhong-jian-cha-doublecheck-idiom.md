# 3. 单例模式与双重检查\(Double-Check idiom\)

　　使用**双重检测**同步延迟加载去创建单例的做法是一个非常优秀的做法，其不但保证了单例，而且切实提高了程序运行效率，对应的代码清单如下：

```java
// 线程安全的懒汉式单例
public class Singleton3 {

    //使用volatile关键字防止重排序，
    //因为 new Instance()是一个非原子操作，可能创建一个不完整的实例
    private static volatile Singleton3 singleton3;

    private Singleton3() {
    }

    public static Singleton3 getSingleton3() {
        // Double-Check idiom
        if (singleton3 == null) {
            synchronized (Singleton3.class) {       // 1
                // 只需在第一次创建实例时才同步
                if (singleton3 == null) {       // 2
                    singleton3 = new Singleton3();      // 3
                }
            }
        }
        return singleton3;
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

　　如上述代码所示，为了在保证单例的前提下提高运行效率，我们需要对 singleton3 进行**第二次检查**，目的是**避开过多的同步（因为这里的同步只需在第一次创建实例时才同步，一旦创建成功，以后获取实例时就不需要同步获取锁了）**。这种做法无疑是优秀的，但是我们必须注意一点：**必须使用volatile关键字修饰单例引用**。

　　**那么，如果上述的实现没有使用 volatile 修饰 singleton3，会导致什么情形发生呢？** 为解释该问题，我们分两步来阐述：

## \(1\)、当我们写了 new 操作，JVM 到底会发生什么？

　　首先，我们要明白的是： **new Singleton3\(\) 是一个非原子操作。**代码行singleton3 = new Singleton3\(\); 的执行过程可以形象地用如下3行伪代码来表示：

```java
memory = allocate();        //1:分配对象的内存空间
ctorInstance(memory);       //2:初始化对象
singleton3 = memory;        //3:使singleton3指向刚分配的内存地址123
```

　　但实际上，这个过程可能发生无序写入\(指令重排序\)，也就是说上面的3行指令可能会被重排序导致先执行第3行后执行第2行，也就是说其真实执行顺序可能是下面这种：

```java
memory = allocate();        //1:分配对象的内存空间
singleton3 = memory;        //3:使singleton3指向刚分配的内存地址
ctorInstance(memory);       //2:初始化对象123
```

　　这段伪代码演示的情况不仅是可能的，而且是一些 JIT 编译器上真实发生的现象。

## \(2\)、重排序情景再现 

        了解 new 操作是非原子的并且可能发生重排序这一事实后，我们回过头看使用 Double-Check idiom 的同步延迟加载的实现：

　　我们需要重新考察上述清单中的 //3 行。此行代码创建了一个 Singleton 对象并初始化变量 singleton3 来引用此对象。这行代码存在的问题是，在 Singleton 构造函数体执行之前，变量 singleton3 可能提前成为非 null 的，即赋值语句在对象实例化之前调用，此时别的线程将得到的是一个不完整（未初始化）的对象，会导致系统崩溃。下面是程序可能的一组执行步骤：

　　1、线程 1 进入 getSingleton3\(\) 方法；   
　　2、由于 singleton3 为 null，线程 1 在 //1 处进入 synchronized 块；   
　　3、同样由于 singleton3 为 null，线程 1 直接前进到 //3 处，但在构造函数执行之前，使实例成为非 null，并且该实例是未初始化的；   
　　4、线程 1 被线程 2 预占；   
　　5、线程 2 检查实例是否为 null。因为实例不为 null，线程 2 得到一个不完整（未初始化）的 Singleton 对象；   
　　6、线程 2 被线程 1 预占。   
　　7、线程 1 通过运行 Singleton3 对象的构造函数来完成对该对象的初始化。

　　显然，一旦我们的程序在执行过程中发生了上述情形，就会造成灾难性的后果，而这种安全隐患正是由于指令重排序的问题所导致的。让人兴奋地是，volatile 关键字正好可以完美解决了这个问题。也就是说，我们只需使用volatile关键字修饰单例引用就可以避免上述灾难。


# 3.1 ThreadLocal

## 1. ThreadLocal的简介

在多线程编程中通常解决线程安全的问题我们会利用synchronzed或者lock控制线程对临界区资源的同步顺序从而解决线程安全的问题，但是这种加锁的方式会让未获取到锁的线程进行阻塞等待，很显然这种方式的时间效率并不是很好。**线程安全问题的核心在于多个线程会对同一个临界区共享资源进行操作**，那么，如果每个线程都使用自己的“共享资源”，各自使用各自的，又互相不影响到彼此即让多个线程间达到隔离的状态，这样就不会出现线程安全的问题。事实上，这就是一种“**空间换时间**”的方案，每个线程都会都拥有自己的“共享资源”无疑内存会大很多，但是由于不需要同步也就减少了线程可能存在的阻塞等待的情况从而提高的时间效率。

虽然**ThreadLocal**并不在java.util.concurrent包中而在java.lang包中，但我更倾向于把它当作是一种并发容器（虽然真正存放数据的是**ThreadLoclMap**）进行归类。从**ThreadLocal这个类名可以顾名思义的进行理解，表示线程的“本地变量”，即每个线程都拥有该变量副本，达到人手一份的效果，各用各的这样就可以避免共享资源的竞争**。

## 2. ThreadLocal的实现原理

要想学习到ThreadLocal的实现原理，就必须了解它的几个核心方法:

### 1. Set\(T value\)

```java
void set(T value)
```

 **set方法设置在当前线程中threadLocal变量的值**，该方法的源码为：

```java
public void set(T value) {
    //1. 获取当前线程实例对象
    Thread t = Thread.currentThread();
    //2. 通过当前线程实例获取到ThreadLocalMap对象
    ThreadLocalMap map = getMap(t);
    if (map != null)
        //3. 如果Map不为null,则以当前threadLocl实例为key,值为value进行存入
        map.set(this, value);
    else
        //4.map为null,则新建ThreadLocalMap并存入value
        createMap(t, value);
}
```

 通过源码我们知道value是存放在了ThreadLocalMap里了，当前先把它理解为一个普普通通的map即可，也就是说，**数据value是真正的存放在了ThreadLocalMap这个容器中了，并且是以当前threadLocal实例为key。**

**首先ThreadLocalMap是怎样来的**？源码很清楚，是通过`getMap(t)`进行获取：

```java
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

该方法直接**返回的就是当前线程对象t的一个成员变量threadLocals**：

```java
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```

也就是说**ThreadLocalMap的引用是作为Thread的一个成员变量，被Thread进行维护的**。回过头再来看看set方法，当map为Null的时候会通过`createMap(t，value)`方法：

```java
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

该方法就是**new一个ThreadLocalMap实例对象，然后同样以当前threadLocal实例作为key,值为value存放到threadLocalMap中，然后将当前线程对象的threadLocals赋值为threadLocalMap**。

现在来对set方法进行总结一下：  
    **通过当前线程对象thread获取该thread所维护的threadLocalMap; 若threadLocalMap不为null,则以threadLocal实例为key,值为value的键值对存入threadLocalMap; 若threadLocalMap为null的话，就新建threadLocalMap然后在以threadLocal为键，值为value的键值对存入即可。**

### 2. T get\(\)

**get方法是获取当前线程中threadLocal变量的值**，同样的还是来看看源码：

```java
public T get() {
    //1. 获取当前线程的实例对象
    Thread t = Thread.currentThread();
    //2. 获取当前线程的threadLocalMap
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        //3. 获取map中当前threadLocal实例为key的值的entry
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            //4. 当前entitiy不为null的话，就返回相应的值value
            T result = (T)e.value;
            return result;
        }
    }
    //5. 若map为null或者entry为null的话通过该方法初始化，并返回该方法返回的value
    return setInitialValue();
}
```

弄懂了set方法的逻辑，看get方法只需要带着逆向思维去看就好，如果是那样存的，反过来去拿就好。代码逻辑请看注释，另外，看下setInitialValue主要做了些什么事情？

```java
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```

这段方法的逻辑和set方法几乎一致，另外值得关注的是initialValue方法:

```java
protected T initialValue() {
    return null;
}
```

这个**方法是protected修饰的也就是说继承ThreadLocal的子类可重写该方法，实现赋值为其他的初始值**。

关于get方法来总结一下：

**通过当前线程thread实例获取到它所维护的threadLocalMap，然后以当前threadLocal实例为key获取该map中的键值对（Entry），若Entry不为null则返回Entry的value。如果获取threadLocalMap为null或者Entry为null的话，就以当前threadLocal为Key，value为null存入map后，并返回null。**

### 3. void remove\(\)

```java
public void remove() {
    //1. 获取当前线程的threadLocalMap
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        //2. 从map中删除以当前threadLocal实例为key的键值对
        m.remove(this);
}
```

get,set方法实现了存数据和读数据，我们当然还得学会如何删数据**。删除数据当然是从map中删除数据，先获取与当前线程相关联的threadLocalMap然后从map中删除该threadLocal实例为key的键值对即可**。

## 


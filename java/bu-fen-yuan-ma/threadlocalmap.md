# ThreadLocalMap （上）

## 1. ThreadLocalMap详解

从上面的分析我们已经知道，数据其实都放在了threadLocalMap中，threadLocal的get，set和remove方法实际上具体是通过threadLocalMap的getEntry,set和remove方法实现的。如果想真正全方位的弄懂threadLocal，势必得在对threadLocalMap做一番理解。

### 1.1 Entry数据结构

ThreadLocalMap是threadLocal一个静态内部类，和大多数容器一样内部维护了一个数组，同样的threadLocalMap内部维护了一个Entry类型的table数组。

```java
/**
 * The table, resized as necessary.
 * table.length MUST always be a power of two.
 */
private Entry[] table;
```

通过注释可以看出，table数组的长度为2的幂次方。接下来看下Entry是什么：

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

Entry是一个以ThreadLocal为key,Object为value的键值对，另外需要注意的是这里的**threadLocal是弱引用，因为Entry继承了WeakReference，在Entry的构造方法中，调用了super\(k\)方法就会将threadLocal实例包装成一个WeakReferenece。**到这里我们可以用一个图来理解下thread,threadLocal,threadLocalMap，Entry之间的关系：

![ThreadLocal&#x5404;&#x5F15;&#x7528;&#x95F4;&#x7684;&#x5173;&#x7CFB;](../../.gitbook/assets/image%20%2874%29.png)

注意上图中的实线表示强引用，虚线表示弱引用。如图所示，每个线程实例中可以通过threadLocals获取到threadLocalMap，而threadLocalMap实际上就是一个以threadLocal实例为key，任意对象为value的Entry数组。当我们为threadLocal变量赋值，实际上就是以当前threadLocal实例为key，值为value的Entry往这个threadLocalMap中存放。需要注意的是**Entry中的key是弱引用，当threadLocal外部强引用被置为null\(`threadLocalInstance=null`\),那么系统 GC 的时候，根据可达性分析，这个threadLocal实例就没有任何一条链路能够引用到它，这个ThreadLocal势必会被回收，这样一来，ThreadLocalMap中就会出现key为null的Entry，就没有办法访问这些key为null的Entry的value，如果当前线程再迟迟不结束的话，这些key为null的Entry的value就会一直存在一条强引用链：Thread Ref -&gt; Thread -&gt; ThreaLocalMap -&gt; Entry -&gt; value永远无法回收，造成内存泄漏。**当然，如果当前thread运行结束，threadLocal，threadLocalMap,Entry没有引用链可达，在垃圾回收的时候都会被系统进行回收。在实际开发中，会使用**线程池**去维护线程的创建和复用，比如**固定大小的线程**池，线程为了复用是不会主动结束的，所以，threadLocal的内存泄漏问题。

### 1.2 set方法

源码为：

```java
private void set(ThreadLocal<?> key, Object value) {

    // We don't use a fast path as with get() because it is at
    // least as common to use set() to create new entries as
    // it is to replace existing ones, in which case, a fast
    // path would fail more often than not.

    Entry[] tab = table;
    int len = tab.length;
    //根据threadLocal的hashCode确定Entry应该存放的位置
    int i = key.threadLocalHashCode & (len-1);

    //采用开放地址法，hash冲突的时候使用线性探测
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        //覆盖旧Entry
        if (k == key) {
            e.value = value;
            return;
        }
        //当key为null时，说明threadLocal强引用已经被释放掉，那么就无法
        //再通过这个key获取threadLocalMap中对应的entry，这里就存在内存泄漏的可能性
        if (k == null) {
            //用当前插入的值替换掉这个key为null的“脏”entry
            replaceStaleEntry(key, value, i);
            return;
        }
    }
    //新建entry并插入table中i处
    tab[i] = new Entry(key, value);
    int sz = ++size;
    //插入后再次清除一些key为null的“脏”entry,如果大于阈值就需要扩容
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```

set方法的关键部分**请看上面的注释**，主要有这样几点需要注意：

1. threadLocal的hashcode?

   ```java
    private final int threadLocalHashCode = nextHashCode();
    private static final int HASH_INCREMENT = 0x61c88647;
    private static AtomicInteger nextHashCode =new AtomicInteger();
    /**
     * Returns the next hash code.
     */
    private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }
   ```

   从源码中我们可以清楚的看到threadLocal实例的hashCode是通过nextHashCode\(\)方法实现的，该方法实际上总是用一个AtomicInteger加上0x61c88647来实现的。0x61c88647这个数是有特殊意义的，它能够保证hash表的每个散列桶能够均匀的分布，这是`Fibonacci Hashing`。也正是能够均匀分布，所以threadLocal选择使用**开放地址法来解决hash冲突**的问题。

2. 怎样确定新值插入到哈希表中的位置？

   该操作源码为：`key.threadLocalHashCode & (len-1)`，同hashMap和ConcurrentHashMap等容器的方式一样，利用当前key\(即threadLocal实例\)的hashcode与哈希表大小相与，因为哈希表大小总是为2的幂次方，所以相与等同于一个取模的过程，这样就可以通过Key分配到具体的哈希桶中去。而至于为什么取模要通过位与运算的原因就是**位运算的执行效率远远高于了取模运算**。

3. 怎样**解决hash冲突**？

   源码中通过`nextIndex(i, len)`方法解决hash冲突的问题，该方法为`((i + 1 < len) ? i + 1 : 0);`，也就是不断**往后线性探测**，当**到哈希表末尾的时候再从0开始，成环形**。

4. 怎样解决“脏”Entry？

   在分析threadLocal,threadLocalMap以及Entry的关系的时候，我们已经知道使用threadLocal有可能存在内存泄漏（对象创建出来后，在之后的逻辑一直没有使用该对象，但是垃圾回收器无法回收这个部分的内存），在源码中针对这种key为null的Entry称之为“stale entry”，直译为不新鲜的entry，我把它理解为“脏entry”，自然而然，Josh Bloch and Doug Lea大师考虑到了这种情况,在set方法的for循环中寻找和当前Key相同的可覆盖entry的过程中通过**replaceStaleEntry**方法解决脏entry的问题。如果当前table\[i\]为null的话，直接插入新entry后也会通过**cleanSomeSlots**来解决脏entry的问题，关于cleanSomeSlots和replaceStaleEntry方法。

5. 如何进行扩容？

> threshold的确定

```java
    private int threshold; // Default to 0
    /**
     * The initial capacity -- MUST be a power of two.
     */
    private static final int INITIAL_CAPACITY = 16;
    
    ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
        table = new Entry[INITIAL_CAPACITY];
        int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
        table[i] = new Entry(firstKey, firstValue);
        size = 1;
        setThreshold(INITIAL_CAPACITY);
    }
    
    /**
     * Set the resize threshold to maintain at worst a 2/3 load factor.
     */
    private void setThreshold(int len) {
        threshold = len * 2 / 3;
    }
```

根据源码可知，在第一次为threadLocal进行赋值的时候会创建初始大小为16的threadLocalMap,并且通过setThreshold方法设置threshold，其值为当前哈希数组长度乘以（2/3），也就是说加载因子为2/3\(**加载因子是衡量哈希表密集程度的一个参数，如果加载因子越大的话，说明哈希表被装载的越多，出现hash冲突的可能性越大，反之，则被装载的越少，出现hash冲突的可能性越小。同时如果过小，很显然内存使用率不高，该值取值应该考虑到内存使用率和hash冲突概率的一个平衡，如hashMap,concurrentHashMap的加载因子都为0.75**\)。这里**threadLocalMap初始大小为16**，**加载因子为2/3**，所以哈希表可用大小为：16\*2/3=10，即哈希表可用容量为10。

### 1.3 resize方法

从set方法中可以看出当hash表的size大于threshold的时候，会通过resize方法进行扩容。

```java
/**
 * Double the capacity of the table.
 */
private void resize() {
    Entry[] oldTab = table;
    int oldLen = oldTab.length;
    //新数组为原数组的2倍
    int newLen = oldLen * 2;
    Entry[] newTab = new Entry[newLen];
    int count = 0;

    for (int j = 0; j < oldLen; ++j) {
        Entry e = oldTab[j];
        if (e != null) {
            ThreadLocal<?> k = e.get();
            //遍历过程中如果遇到脏entry的话直接另value为null,有助于value能够被回收
            if (k == null) {
                e.value = null; // Help the GC
            } else {
                //重新确定entry在新数组的位置，然后进行插入
                int h = k.threadLocalHashCode & (newLen - 1);
                while (newTab[h] != null)
                    h = nextIndex(h, newLen);
                newTab[h] = e;
                count++;
            }
        }
    }
    //设置新哈希表的threshHold和size属性
    setThreshold(newLen);
    size = count;
    table = newTab;
}   
```

方法逻辑**请看注释**，新建一个大小为原来数组长度的两倍的数组，然后遍历旧数组中的entry并将其插入到新的hash数组中，主要注意的是，**在扩容的过程中针对脏entry的话会令value为null，以便能够被垃圾回收器能够回收，解决隐藏的内存泄漏的问题**。

### 1.4 getEntry方法

getEntry方法源码为：

```java
private Entry getEntry(ThreadLocal<?> key) {
    //1. 确定在散列数组中的位置
    int i = key.threadLocalHashCode & (table.length - 1);
    //2. 根据索引i获取entry
    Entry e = table[i];
    //3. 满足条件则返回该entry
    if (e != null && e.get() == key)
        return e;
    else
        //4. 未查找到满足条件的entry，额外在做的处理
        return getEntryAfterMiss(key, i, e);
}
```

方法逻辑很简单，若能当前定位的entry的key和查找的key相同的话就直接返回这个entry，否则的话就是在set的时候存在hash冲突的情况，需要通过getEntryAfterMiss做进一步处理。getEntryAfterMiss方法为：

```java
private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            //找到和查询的key相同的entry则返回
            return e;
        if (k == null)
            //解决脏entry的问题
            expungeStaleEntry(i);
        else
            //继续向后环形查找
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```

这个方法同样很好理解，通过nextIndex往后环形查找，如果找到和查询的key相同的entry的话就直接返回，如果在查找过程中遇到脏entry的话使用expungeStaleEntry方法进行处理。到目前为止**，为了解决潜在的内存泄漏的问题，在set，resize, getEntry这些地方都会对这些脏entry进行处理，可见为了尽可能解决这个问题几乎无时无刻都在做出努力。**

### 1.5 remove

```java
/**
 * Remove the entry for key.
 */
private void remove(ThreadLocal<?> key) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        if (e.get() == key) {
            //将entry的key置为null
            e.clear();
            //将该entry的value也置为null
            expungeStaleEntry(i);
            return;
        }
    }
}
```

该方法逻辑很简单，通过往后环形查找到与指定key相同的entry后，先**通过clear方法将key置为null后，使其转换为一个脏entry，然后调用expungeStaleEntry方法将其value置为null**，以便垃圾回收时能够清理，同时将table\[i\]置为null。

## 2. ThreadLocal的使用场景

        **ThreadLocal 不是用来解决共享对象的多线程访问问题的**，数据实质上是放在每个thread实例引用的threadLocalMap,也就是说**每个不同的线程都拥有专属于自己的数据容器（threadLocalMap），彼此不影响**。因此threadLocal只适用于 **共享对象会造成线程安全** 的业务场景。比如**hibernate中通过threadLocal管理Session**就是一个典型的案例，不同的请求线程（用户）拥有自己的session,若将session共享出去被多线程访问，必然会带来线程安全问题。下面，我们自己来写一个例子，SimpleDateFormat.parse方法会有线程安全的问题，我们可以尝试使用threadLocal包装SimpleDateFormat，将该实例不被多线程共享即可。

```java
public class ThreadLocalDemo {
    private static ThreadLocal<SimpleDateFormat> sdf = new ThreadLocal<>();

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 100; i++) {
            executorService.submit(new DateUtil("2019-11-25 09:00:" + i % 60));
        }
    }

    static class DateUtil implements Runnable {
        private String date;

        public DateUtil(String date) {
            this.date = date;
        }

        @Override
        public void run() {
            if (sdf.get() == null) {
                sdf.set(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
            } else {
                try {
                    Date date = sdf.get().parse(this.date);
                    System.out.println(date);
                } catch (ParseException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

1. 如果当前线程不持有SimpleDateformat对象实例，那么就新建一个并把它设置到当前线程中，如果已经持有，就直接使用。
2. 另外，**从`if (sdf.get() == null){....}else{.....}`可以看出为每一个线程分配一个SimpleDateformat对象实例是从应用层面（业务代码逻辑）去保证的。**
3. 在上面我们说过threadLocal有可能存在内存泄漏，在使用完之后，最好使用remove方法将这个变量移除，就像在使用数据库连接一样，及时关闭连接。




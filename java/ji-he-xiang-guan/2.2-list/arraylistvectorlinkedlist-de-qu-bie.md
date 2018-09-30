# ArrayList、Vector、LinkedList 的区别

## 1. 同步问题

        ArrayList, LinkedList是不同步的，而Vector是同步的（多线程）；当然，也可以通过一些办法包装ArrayList, LinkedList，使他们也达到同步，但效率可能会有所降低 , Vector 有一个Stack子类。  
        实现：**Synchronized 关键字** 修饰大多数关键方法，没有使用volatile关键字。

## 2. 时间空间问题

###  1. 继承方式不同

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

```java
public class Vector<E>
            extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

```java
public class LinkedList<E>
            extends AbstractSequentialList<E>
        implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

1. ArrayList和Vector继承方式**相同**，而LinkedList与上述两个**不同**；
2. 实现了Serializable接口，因此它支持序列化，能够通过序列化传输；
3. 实现了**RandomAccess接口，支持快速随机访问**，实际上就是通过下标进行快速访问；
4. 实现了Cloneable接口，能被克隆。

### 2. 数据增长

####         1. ArrayList和Vector都是使用**Objec的数组形式（需要扩容）**来存储的；

```java
     transient Object[] elementData;              // ArryList
         
     protected Object[] elementData;              // Vector
         
     private static class Node<E> {               //LinkedList
            E item;
            Node<E> next;
            Node<E> prev;

            Node(Node<E> prev, E element, Node<E> next) {
                this.item = element;
                this.next = next;
                this.prev = prev;
            }
        }
```

        动态扩容：如果元素的数目超出了**内部数组**目前的长度它们都需要扩展内部数组的长度；**Vector缺省情况下自动增长原来一倍的数组长度，ArrayList是原来的50%**, 所以最后你获得的这个集合所占的空间总是比你实际需要的要大。

        ArrayList的动态扩容在添加元素时：

```java
public boolean add(E e) {
                 //确保内部容量(通过判断，如果够则不进行操作；容量不够就扩容来确保内部容量)
            ensureCapacityInternal(size + 1);               // ①Increments modCount!!
            elementData[size++] = e;                        //②
             return true;
       }
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;                                //private static final int DEFAULT_CAPACITY = 10;
        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
  
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);   // 如果实际存储数组 是空数组，则最小需要容量就是默认容量
 
        }

        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)            //如果数组（elementData)的长度小于最小需要的容量（minCapacity）就扩容
            grow(minCapacity);

    }
                                                                      //扩容算法
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);           // 带符号右移动一位，所以，为1.5倍
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;                                //扩容1.5倍还不够，直接等于它
        if (newCapacity - MAX_ARRAY_SIZE > 0)                         //如果大于最大的空间 [0x7ffffff7]
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?                         //MAX_ARRAY_SIZE==[0x7ffffff7]
            Integer.MAX_VALUE :                                         //Integer.MAX_VALUE=[0x7fffffff]
            MAX_ARRAY_SIZE;
    }
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;         
//减去8是因为OutOfMemoryError:    Requested array size exceeds VM limit（ Some VMs reserve some header words in an array.）
```

        Vector的动态扩容：

```java
 public synchronized boolean add(E e) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }
public synchronized void ensureCapacity(int minCapacity) {
        if (minCapacity > 0) {
            modCount++;
            ensureCapacityHelper(minCapacity);
        }
    }

    /**
     * This implements the unsynchronized semantics of ensureCapacity.
     * Synchronized methods in this class can internally call this
     * method for ensuring capacity without incurring the cost of an
     * extra synchronization.
     *
     * @see #ensureCapacity(int)
     */
    private void ensureCapacityHelper(int minCapacity) {
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /**
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

    **扩容都是通过Arrays.copyOf\(elementData, newCapacity\) 这样的方式实现的。**

####        2. 在LinkedList中有一个私有的内部类，定义如下： 

```java
     private static class Entry {   
             Object element;   
             Entry next;   
             Entry previous;   
          }   
```

        每个Entry对象 reference列表中的一个元素，同时还有在LinkedList中它的上一个元素和下一个元素。一个有1000个元素的LinkedList对象将 有1000个链接在一起的Entry对象，每个对象都对应于列表中的一个元素。这样的话，**在一个LinkedList结构中将有一个很大的空间开销，因为 它要存储这1000个Entity对象的相关信息**。 

        ArrayList使用一个内置的数组来存储元素，这个数组的起始容量是10.当数组需要增长时，新的容量按 如下公式获得：**新容量=\(旧容量\*3\)/2+1，也就是说每一次容量大概会增长50%。这就意味着，如果你有一个包含大量元素的ArrayList对象， 那么最终将有很大的空间会被浪费掉**，这个浪费是由ArrayList的工作方式本身造成的。如果没有足够的空间来存放新的元素，数组将不得不被重新进行分 配以便能够增加新的元素。对数组进行重新分配，将会导致性能急剧下降。**如果我们知道一个ArrayList将会有多少个元素，我们可以通过构造方法来指定容量。我们还可以通过trimToSize方法在ArrayList和Vector分配完毕之后去掉浪费掉的空间。**

    trimToSize方法：

```java
public void trimToSize() {                    //ArryList
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0) 
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
              //size为List的大小，而elementData.length为内部数组长度
        }
    }
```

```java
public synchronized void trimToSize() {               //Vector
        modCount++;
        int oldCapacity = elementData.length;         
        if (elementCount < oldCapacity) {
            elementData = Arrays.copyOf(elementData, elementCount);
        }
        //elementCount为List的大小，而elementData.length为内部数组长度
    }
```

### 4. 增删改查

        ArrayList和Vector中，从指定的位置（用index）**查找、修改**一个对象，或在集合的**末尾插入、删除**一个对象的时间是一样的，可表示为**O\(1\)**。但是，如果在集合的其他位置**增加**或**移除**元素那么花费的时间会呈线形增长：**O\(n-i\)**，其中n代表集合中元素的个数，i代表元素增加或移除元素的索引位置。

        LinkedList中，在**插入、删除**集合中任何位置的元素所花费的时间都是一样的—**O\(1\)**，但它在**查找**一个元素的时候比较慢，为O\(i\),其中i是索引的位置。 

       总结：

              **LinkedList                      增删：O\(1\)        改查：O\(i\)（i是索引的位置）**

              **ArrayList和Vector         增删：O\(n-i\)     改查：O\(1\)**






# 2. Buffer

## 1. Buffer

> A buffer is a linear, finite sequence of elements of a specific primitive type.

一块缓存区，内部使用**字节数组**存储数据，并维护几个特殊变量，实现数据的反复利用。  
 1、**mark**：初始值为-1，用于备份当前的position;  
 2、**position**：初始值为0，position表示当前可以写入或读取数据的位置，当写入或读取一个数据后，position向前移动到下一个位置；  
 3、**limit**：写模式下，limit表示最多能往Buffer里写多少数据，等于capacity值；读模式下，limit表示最多可以读取多少数据。  
 4、**capacity**：缓存数组大小

![](../../../.gitbook/assets/image%20%28329%29.png)

### **mark\(\)**：

把当前的position赋值给mark

```java
public final Buffer mark() {
    mark = position;
    return this;
}
```

### **reset\(\)**：

把mark值还原给position

```java
public final Buffer reset() {
    int m = mark;
    if (m < 0)
        throw new InvalidMarkException();
    position = m;
    return this;
}
```

### **clear\(\)**：

一旦读完Buffer中的数据，需要让Buffer准备好再次被写入，clear会恢复状态值，但不会擦除数据。

```java
public final Buffer clear() {
    position = 0;
    limit = capacity;
    mark = -1;
    return this;
}
```

### **flip\(\)**：

Buffer有两种模式，写模式和读模式，flip后Buffer从写模式变成读模式。

```java
public final Buffer flip() {
    limit = position;
    position = 0;
    mark = -1;
    return this;
}
```

### **rewind\(\)**：

重置position为0，从头读写数据。

```java
public final Buffer rewind() {
    position = 0;
    mark = -1;
    return this;
}
```

## 2. 分类

目前Buffer的实现类有以下几种：

* ByteBuffer
* CharBuffer
* DoubleBuffer
* FloatBuffer
* IntBuffer
* LongBuffer
* ShortBuffer
* MappedByteBuffer

![](../../../.gitbook/assets/image%20%28172%29.png)

## 3. ByteBuffer

> A byte buffer，extend from Buffer

ByteBuffer的实现类包括"HeapByteBuffer"和"DirectByteBuffer"两种。

### **HeapByteBuffer**

```java
public static ByteBuffer allocate(int capacity) {
    if (capacity < 0)
        throw new IllegalArgumentException();
    return new HeapByteBuffer(capacity, capacity);
}
HeapByteBuffer(int cap, int lim) {  
    super(-1, 0, lim, cap, new byte[cap], 0);
}
```

HeapByteBuffer通过初始化字节数组hd，在虚拟机堆上申请内存空间。

### **DirectByteBuffer**

```java
public static ByteBuffer allocateDirect(int capacity) {
    return new DirectByteBuffer(capacity);
}
DirectByteBuffer(int cap) {
    super(-1, 0, cap, cap);
    boolean pa = VM.isDirectMemoryPageAligned();
    int ps = Bits.pageSize();
    long size = Math.max(1L, (long)cap + (pa ? ps : 0));
    Bits.reserveMemory(size, cap);

    long base = 0;
    try {
        base = unsafe.allocateMemory(size);
    } catch (OutOfMemoryError x) {
        Bits.unreserveMemory(size, cap);
        throw x;
    }
    unsafe.setMemory(base, size, (byte) 0);
    if (pa && (base % ps != 0)) {
        // Round up to page boundary
        address = base + ps - (base & (ps - 1));
    } else {
        address = base;
    }
    cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
    att = null;
}
```

DirectByteBuffer通过unsafe.allocateMemory申请**堆外内存**，并在ByteBuffer的address变量中维护指向该内存的地址。  
 unsafe.setMemory\(base, size, \(byte\) 0\)方法把新申请的内存数据清零。  
  



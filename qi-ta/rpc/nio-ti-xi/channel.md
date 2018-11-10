# 3. Channel

## 1. 概述

> A channel represents an open connection to an entity such as a hardware device, a file, a network socket, or a program component that is capable of performing one or more distinct I/O operations, for example reading or writing.

NIO把它支持的I/O对象抽象为**Channel**，Channel又称“通道”，类似于原I/O中的流（Stream），但有所区别：  
 1、流是**单向**的，通道是**双向**的，可读可写。  
 2、流读写是**阻塞**的，通道可以**异步**读写。  
 3、流中的数据可以选择性的先读到缓存中，通道的数据总是要先读到一个**缓存**中，或从缓存中写入，如下所示：

![](../../../.gitbook/assets/image%20%28279%29.png)

## 2. 分类

目前已知Channel的实现类有：

* FileChannel
* DatagramChannel
* SocketChannel
* ServerSocketChannel

## **2. FileChannel**

> A channel for reading, writing, mapping, and manipulating a file.  
>  一个用来写、读、映射和操作文件的通道。

FileChannel的read、write和map通过其实现类FileChannelImpl实现。

### **read实现**

```java
public int read(ByteBuffer dst) throws IOException {
    ensureOpen();
    if (!readable)
        throw new NonReadableChannelException();
    synchronized (positionLock) {
        int n = 0;
        int ti = -1;
        try {
            begin();
            ti = threads.add();
            if (!isOpen())
                return 0;
            do {
                n = IOUtil.read(fd, dst, -1, nd);
            } while ((n == IOStatus.INTERRUPTED) && isOpen());
            return IOStatus.normalize(n);
        } finally {
            threads.remove(ti);
            end(n > 0);
            assert IOStatus.check(n);
        }
    }
}
```

FileChannelImpl的**read方法**通过IOUtil的read实现：

```java
static int read(FileDescriptor fd, ByteBuffer dst, long position,
                NativeDispatcher nd) IOException {
    if (dst.isReadOnly())
        throw new IllegalArgumentException("Read-only buffer");
    if (dst instanceof DirectBuffer)
        return readIntoNativeBuffer(fd, dst, position, nd);

    // Substitute a native buffer
    ByteBuffer bb = Util.getTemporaryDirectBuffer(dst.remaining());
    try {
        int n = readIntoNativeBuffer(fd, bb, position, nd);
        bb.flip();
        if (n > 0)
            dst.put(bb);
        return n;
    } finally {
        Util.offerFirstTemporaryDirectBuffer(bb);
    }
}
```

通过上述实现可以看出，基于channel的文件数据读取步骤如下：  
 1、申请一块和缓存同大小的DirectByteBuffer bb。  
 2、读取数据到缓存bb，底层由NativeDispatcher的read实现。  
 3、把bb的数据读取到dst（用户定义的缓存，在jvm中分配内存）。  
 **read方法导致数据复制了两次**。

### **write实现**

```java
public int write(ByteBuffer src) throws IOException {
    ensureOpen();
    if (!writable)
        throw new NonWritableChannelException();
    synchronized (positionLock) {
        int n = 0;
        int ti = -1;
        try {
            begin();
            ti = threads.add();
            if (!isOpen())
                return 0;
            do {
                n = IOUtil.write(fd, src, -1, nd);
            } while ((n == IOStatus.INTERRUPTED) && isOpen());
            return IOStatus.normalize(n);
        } finally {
            threads.remove(ti);
            end(n > 0);
            assert IOStatus.check(n);
        }
    }
}
```

和read实现一样，FileChannelImpl的write方法通过IOUtil的write实现：

```java
static int write(FileDescriptor fd, ByteBuffer src, long position,
                 NativeDispatcher nd) throws IOException {
    if (src instanceof DirectBuffer)
        return writeFromNativeBuffer(fd, src, position, nd);
    // Substitute a native buffer
    int pos = src.position();
    int lim = src.limit();
    assert (pos <= lim);
    int rem = (pos <= lim ? lim - pos : 0);
    ByteBuffer bb = Util.getTemporaryDirectBuffer(rem);
    try {
        bb.put(src);
        bb.flip();
        // Do not update src until we see how many bytes were written
        src.position(pos);
        int n = writeFromNativeBuffer(fd, bb, position, nd);
        if (n > 0) {
            // now update src
            src.position(pos + n);
        }
        return n;
    } finally {
        Util.offerFirstTemporaryDirectBuffer(bb);
    }
}
```

通过上述实现可以看出，基于channel的文件数据写入步骤如下：  
 1、申请一块DirectByteBuffer，bb大小为byteBuffer中的limit - position。  
 2、复制byteBuffer中的数据到bb中。  
 3、把数据从bb中写入到文件，底层由NativeDispatcher的write实现，具体如下：

```java
private static int writeFromNativeBuffer(FileDescriptor fd, 
        ByteBuffer bb, long position, NativeDispatcher nd)
    throws IOException {
    int pos = bb.position();
    int lim = bb.limit();
    assert (pos <= lim);
    int rem = (pos <= lim ? lim - pos : 0);

    int written = 0;
    if (rem == 0)
        return 0;
    if (position != -1) {
        written = nd.pwrite(fd,
                            ((DirectBuffer)bb).address() + pos,
                            rem, position);
    } else {
        written = nd.write(fd, ((DirectBuffer)bb).address() + pos, rem);
    }
    if (written > 0)
        bb.position(pos + written);
    return written;
}
```

write方法也导致了数据复制了两次

## Channel和Buffer示例

```java
File file = new RandomAccessFile("data.txt", "rw");
FileChannel channel = file.getChannel();
ByteBuffer buffer = ByteBuffer.allocate(48);

int bytesRead = channel.read(buffer);
while (bytesRead != -1) {
    System.out.println("Read " + bytesRead);
    buffer.flip();
    while(buffer.hasRemaining()){
        System.out.print((char) buffer.get());
    }
    buffer.clear();
    bytesRead = channel.read(buffer);
}
file.close();
```

注意buffer.flip\(\) 的调用，首先**将数据写入到buffer，然后变成读模式，再从buffer中读取数据。**


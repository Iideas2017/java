# 参考资料

{% embed data="{\"url\":\"https://blog.csdn.net/quinnnorris/article/details/75040538\",\"type\":\"link\",\"title\":\"java 对象存活分析——引用计数法&可达性分析 - CSDN博客\",\"description\":\"java虚拟机总共分为五个区域，其中三个是线程私有：程序计数器，虚拟机栈，本地方法栈，两个是线程共享：堆，方法区。线程私有的区域等到线程结束时（栈帧出栈时）会自动被释放，空间比较容易清理。而线程共享的java堆和方法区中的空间较大而且没有线程的回收容易产生很多垃圾信息，GC垃圾回收真正关心的就是这部分。java堆和方法区主要存放各种类型的对象（方法区中也存储一些静态变量和全局常量等信息），那么我们在\",\"icon\":{\"type\":\"icon\",\"url\":\"https://csdnimg.cn/public/favicon.ico\",\"aspectRatio\":0}}" %}

{% embed data="{\"url\":\"https://www.cnblogs.com/E-star/p/5556188.html\",\"type\":\"link\",\"title\":\"JVM 新生代老年代 - E\_star - 博客园\",\"description\":\"1.为什么会有年轻代 我们先来屡屡，为什么需要把堆分代？不分代不能完成他所做的事情么？其实不分代完全可以，分代的唯一理由就是优化GC性能。你先想想，如果没有分代，那我们所有的对象都在一块，GC的时候我\",\"icon\":{\"type\":\"icon\",\"url\":\"https://www.cnblogs.com/favicon.ico\",\"aspectRatio\":0}}" %}

{% embed data="{\"url\":\"https://blog.csdn.net/aijiudu/article/details/72991993\",\"type\":\"link\",\"title\":\"JVM架构和GC垃圾回收机制\(JVM面试不用愁\) - CSDN博客\",\"description\":\"JVM架构和GC垃圾回收机制 JVM架构图分析  JVM被分为三个主要的子系统： 1.  类加载器子系统 2.  运行时数据区 3.  执行引擎 1. 类加载器子系统 Java的动态类加载功能是由类加载器子系统处理。当它在运行时（不是编译时）首次引用一个类时，它加载、链接并初始化该类文件。 1.1 加载 类由此组件加载。启动类加载器 \(BootStrap class Loa\",\"icon\":{\"type\":\"icon\",\"url\":\"https://csdnimg.cn/public/favicon.ico\",\"aspectRatio\":0}}" %}

{% embed data="{\"url\":\"https://www.cnblogs.com/chengxuyuanzhilu/p/7088316.html\",\"type\":\"link\",\"title\":\"JVM\(HotSpot\) 7种垃圾收集器的特点及使用场景 - guangboyuan.cn - 博客园\",\"description\":\"这里讨论的收集器基于JDK1.7Update 14之后的HotSpot虚拟机，这个虚拟机包含的所有收集器如下图3-5所示： 上图展示了7种作用于不同分代的收集器，如果两个收集器之间存在连线，就说明它们\",\"icon\":{\"type\":\"icon\",\"url\":\"https://www.cnblogs.com/favicon.ico\",\"aspectRatio\":0}}" %}


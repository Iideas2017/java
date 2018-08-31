# Table of contents

* [java](README.md)

## Java基础知识点总结

* [Java基础相关](java-ji-chu-zhi-shi-dian-zong-jie/java-ji-chu-xiang-guan/README.md)
  * [String、StringBuilder、StringBuffer区别](java-ji-chu-zhi-shi-dian-zong-jie/java-ji-chu-xiang-guan/stringstringbuilderstringbuffer-qu-bie.md)
  * [数据类型](java-ji-chu-zhi-shi-dian-zong-jie/java-ji-chu-xiang-guan/untitled-1.md)
  * [Java 中“==”和equals（Object obj）](java-ji-chu-zhi-shi-dian-zong-jie/java-ji-chu-xiang-guan/java-zhong-he-equalsobject-obj.md)
  * [String 不可变类](java-ji-chu-zhi-shi-dian-zong-jie/java-ji-chu-xiang-guan/untitled.md)
* [Java集合相关](java-ji-chu-zhi-shi-dian-zong-jie/ji-he-xiang-guan/README.md)
  * [Java集合框架](java-ji-chu-zhi-shi-dian-zong-jie/ji-he-xiang-guan/ji-he-kuang-jia.md)
  * [ArrayList、Vector、LinkedList 的区别](java-ji-chu-zhi-shi-dian-zong-jie/ji-he-xiang-guan/arraylistvectorlinkedlist-de-qu-bie.md)
  * [HashMap](java-ji-chu-zhi-shi-dian-zong-jie/ji-he-xiang-guan/hashmap.md)

## Java 部分源码学习

* [String源码学习](java-source-code/string-yuan-ma-xue-xi.md)
* [HashMap源码学习](java-source-code/hashmap-yuan-ma-xue-xi.md)

## JVM相关

* [JVM对字符串变量的处理](jvm-xiang-guan/jvm-dui-zi-fu-chuan-bian-liang-de-chu-li.md)
* [Java中的常量池](jvm-xiang-guan/java-zhong-de-chang-liang-chi/README.md)
  * [8种基本数据类型](jvm-xiang-guan/java-zhong-de-chang-liang-chi/8-zhong-ji-ben-shu-ju-lei-xing.md)
  * [String 类](jvm-xiang-guan/java-zhong-de-chang-liang-chi/string-lei.md)
* [参考资料](jvm-xiang-guan/can-kao-zi-liao.md)

## JVM

* [JVM概述](jvm/jvm-gai-shu.md)
* [1. 类加载器子系统](jvm/1.-lei-jia-zai-qi-zi-xi-tong/README.md)
  * [1.1类加载器与双亲委派模型](jvm/1.-lei-jia-zai-qi-zi-xi-tong/1.1-lei-jia-zai-qi-yu-shuang-qin-wei-pai-mo-xing.md)
  * [1.2类加载机制的破坏](jvm/1.-lei-jia-zai-qi-zi-xi-tong/1.2-lei-jia-zai-ji-zhi-de-po-huai.md)
  * [1.3 Tomcat类加载机制](jvm/1.-lei-jia-zai-qi-zi-xi-tong/untitled.md)
  * [参考资料](jvm/1.-lei-jia-zai-qi-zi-xi-tong/can-kao-zi-liao.md)
* [2. 运行时数据区](jvm/2.-yun-hang-shi-shu-ju-qu.md)
* [3. 执行引擎](jvm/3.-zhi-hang-yin-qing/README.md)
  * [3.1垃圾回收器](jvm/3.-zhi-hang-yin-qing/3.1-la-ji-hui-shou-qi/README.md)
    * [3.1.1 java 对象存活分析](jvm/3.-zhi-hang-yin-qing/3.1-la-ji-hui-shou-qi/3.1.1-java-dui-xiang-cun-huo-fen-xi.md)
    * [3.1.2 对象分配](jvm/3.-zhi-hang-yin-qing/3.1-la-ji-hui-shou-qi/3.1.2-dui-xiang-fen-pei.md)
    * [3.1.3 三种基本的GC算法](jvm/3.-zhi-hang-yin-qing/3.1-la-ji-hui-shou-qi/3.1.3-san-zhong-ji-ben-de-gc-suan-fa.md)
    * [3.1.4 JVM垃圾回收器](jvm/3.-zhi-hang-yin-qing/3.1-la-ji-hui-shou-qi/3.1.4-jvm-la-ji-hui-shou-qi.md)
  * [参考资料](jvm/3.-zhi-hang-yin-qing/untitled.md)
* [并发相关](bing-fa-xiang-guan/README.md)
  * [1. 基础知识](bing-fa-xiang-guan/ji-chu-zhi-shi/README.md)
    * [1.1 线程的状态转换](bing-fa-xiang-guan/ji-chu-zhi-shi/5.-xian-cheng-de-zhuang-tai-zhuan-huan-yi-ji-ji-ben-cao-zuo.md)
    * [1.2 两大核心](bing-fa-xiang-guan/ji-chu-zhi-shi/untitled.md)
    * [1.3 原子、可见、有序性](bing-fa-xiang-guan/ji-chu-zhi-shi/1.-yuan-zi-xing-ke-jian-xing-you-xu-xing.md)
    * [1.4 volatile 变量](bing-fa-xiang-guan/ji-chu-zhi-shi/volatile-bian-liang.md)
    * [1.5 Synchronized 关键字](bing-fa-xiang-guan/ji-chu-zhi-shi/4.-synchronized.md)
    * [1.6 final关键字](bing-fa-xiang-guan/ji-chu-zhi-shi/6.-final-guan-jian-zi.md)
  * [2. Lock相关](bing-fa-xiang-guan/lock-xiang-guan/README.md)
    * [2.1 Lock](bing-fa-xiang-guan/lock-xiang-guan/lock.md)
    * [2.2 AQS](bing-fa-xiang-guan/lock-xiang-guan/aqs.md)
    * [2.3 ReentrantLock](bing-fa-xiang-guan/lock-xiang-guan/reentrantlock.md)
    * [2.4 ReentrantReadWriteLock](bing-fa-xiang-guan/lock-xiang-guan/reentrantreadwritelock.md)
    * [2.5 Condition](bing-fa-xiang-guan/lock-xiang-guan/2.5-condition.md)
  * [参考资料](bing-fa-xiang-guan/can-kao-zi-liao.md)
* [Untitled](untitled.md)


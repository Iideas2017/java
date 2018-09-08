# 2.1 Lock

## 1. concurrent包的结构层次

![concurrent&#x76EE;&#x5F55;&#x7ED3;&#x6784;](../../.gitbook/assets/image%20%2889%29.png)

![concurrent&#x5305;&#x5B9E;&#x73B0;&#x6574;&#x4F53;&#x793A;&#x610F;&#x56FE;](../../.gitbook/assets/image%20%2826%29.png)

## 2. lock简介

### 2.1 Lock接口API

```java
void lock();                                       //获取锁
void lockInterruptibly() throws InterruptedException；//获取锁的过程能够响应中断
boolean tryLock();//非阻塞式响应中断能立即返回，获取锁放回true反之返回fasle
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;//超时获取锁，在超时内或者未中断的情况下能够获取锁
Condition newCondition();//获取与lock绑定的等待通知组件，当前线程必须获得了锁才能进行等待，进行等待时会先释放锁，当再次获取锁时才能从等待中返回
```




# 3. Redis 数据库

## 1. redis单线程问题

         单线程指的是网络请求模块使用了一个线程（所以不需考虑并发安全性），即一个线程处理所有网络请求，其他模块仍用了多个线程。

## 2. 快速执行

\(1\) 绝大部分请求是纯粹的内存操作（非常快速）

\(2\) 采用单线程,避免了不必要的上下文切换和竞争条件

\(3\) 非阻塞IO - IO多路复用

## 3.  redis的内部实现

      内部实现采用epoll，采用了epoll+自己实现的简单的事件框架。epoll中的读、写、关闭、连接都转化成了事件，然后利用epoll的多路复用特性，绝不在io上浪费一点时间 这3个条件不是相互独立的，特别是第一条，如果请求都是耗时的，采用单线程吞吐量及性能可想而知了。应该说redis为特殊的场景选择了合适的技术方案。

## 4. 线程安全问题

        redis实际上是采用了线程封闭的观念，把任务封闭在一个线程，自然避免了线程安全问题，不过对于需要依赖多个redis操作的复合操作来说，依然需要锁，而且有可能是分布式锁。



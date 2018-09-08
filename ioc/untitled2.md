# 参考资料

{% embed data="{\"url\":\"https://www.jianshu.com/p/b7911e0394b0\",\"type\":\"link\",\"title\":\"联合索引的最左前缀匹配原则\",\"description\":\"上表中有一个联合索引，下面开始验证最左匹配原则。当存在username时会使用索引查询： 当没有username时，不会使用索引查询： 当有username，但顺序乱序时也可以使用索引： 在最左...\",\"icon\":{\"type\":\"icon\",\"url\":\"https://cdn2.jianshu.io/assets/apple-touch-icons/152-bf209460fc1c17bfd3e2b84c8e758bc11ca3e570fd411c3bbd84149b97453b99.png\",\"width\":152,\"height\":152,\"aspectRatio\":1}}" %}

{% embed data="{\"url\":\"https://blog.csdn.net/plg17/article/details/78758593\",\"type\":\"link\",\"title\":\"图解MySQL 内连接、外连接、左连接、右连接、全连接……太多了 - CSDN博客\",\"description\":\"用两个表（a\_table、b\_table），关联字段a\_table.a\_id和b\_table.b\_id来演示一下MySQL的内连接、外连接（ 左\(外\)连接、右\(外\)连接、全\(外\)连接）。     MySQL版本：Server version: 5.6.31 MySQL Community Server \(GPL\) 数据库表：a\_table、b\_table 主题：内连接、左连接（左外连\",\"icon\":{\"type\":\"icon\",\"url\":\"https://csdnimg.cn/public/favicon.ico\",\"aspectRatio\":0}}" %}

{% embed data="{\"url\":\"https://www.jianshu.com/p/4dbbaaa200c4\",\"type\":\"link\",\"title\":\"数据库索引为什么使用B+树？\",\"description\":\"概述 B tree：  二叉树（Binary tree），每个节点只能存储一个数。B-tree：B树（B-Tree，并不是B“减”树，横杠为连接符，容易被误导）B树属于多叉树又名平衡多路查找树。...\",\"icon\":{\"type\":\"icon\",\"url\":\"https://cdn2.jianshu.io/assets/apple-touch-icons/152-bf209460fc1c17bfd3e2b84c8e758bc11ca3e570fd411c3bbd84149b97453b99.png\",\"width\":152,\"height\":152,\"aspectRatio\":1}}" %}

{% embed data="{\"url\":\"https://blog.csdn.net/stu\_hsj/article/details/46603681\",\"type\":\"link\",\"title\":\"数据库的脏读、不可重复读、幻读以及不可重复读和幻读的区别 - CSDN博客\",\"description\":\"介绍数据库的脏读、不可重复读、幻读都和事务的隔离性有关。所以先了解一下事务的4大特性。  事务的4大特性（ACID）：原子性\(Atomicity\)：事务是数据库的逻辑工作单位，它对数据库的修改要么全部执行，要么全部不执行。 一致性\(Consistemcy\)：事务前后，数据库的状态都满足所有的完整性约束。 隔离性\(Isolation\)：并发执行的N个事务是隔离的，一个不影响一个，一个事务在没有comm\",\"icon\":{\"type\":\"icon\",\"url\":\"https://csdnimg.cn/public/favicon.ico\",\"aspectRatio\":0}}" %}

{% embed data="{\"url\":\"https://blog.csdn.net/yuxin6866/article/details/52649048\",\"type\":\"link\",\"title\":\"对于脏读，不可重复读，幻读的一点理解，看懂红字很关键 - CSDN博客\",\"description\":\"事务4个隔离界别 Read Uncommitted, Read commited, Repeatable read, Serializable Read Uncommitted.  最低的隔离级别，Read Uncommitted最直接的效果就是一个事务可以读取另一个事务并未提交的更新结果。    Read Committed.  Read Committed通常是大部分数据库采用的默\",\"icon\":{\"type\":\"icon\",\"url\":\"https://csdnimg.cn/public/favicon.ico\",\"aspectRatio\":0}}" %}

{% embed data="{\"url\":\"https://blog.csdn.net/bigtree\_3721/article/details/73151472\",\"type\":\"link\",\"title\":\"数据库为什么要用B+树结构- - CSDN博客\",\"description\":\"B+树在数据库中的应用 { 为什么使用B+树？言简意赅，就是因为： 1.文件很大，不可能全部存储在内存中，故要存储到磁盘上 2.索引的结构组织要尽量减少查找过程中磁盘I/O的存取次数（为什么使用B-/+Tree，还跟磁盘存取原理有关。） 3.局部性原理与磁盘预读，预读的长度一般为页（page）的整倍数，（在许多操作系统中，页得大小通常为4k） 4.数据库系统巧妙利用了磁盘预读原理，将一\",\"icon\":{\"type\":\"icon\",\"url\":\"https://csdnimg.cn/public/favicon.ico\",\"aspectRatio\":0}}" %}

{% embed data="{\"url\":\"https://blog.csdn.net/hcmony/article/details/77850183\",\"type\":\"link\",\"title\":\"Spring事务管理与传播机制详解以及使用实例 - CSDN博客\",\"description\":\"写这篇博客之前我首先读了《spring in action》，之后在网上看了一些关于Spring事务管理的文章，感觉都没有讲全，这里就将书上的和网上关于事务的知识总结一下，参考的文章如下：   Spring事务机制详解Spring事务配置的五种方式Spring中的事务管理实例详解   1 初步理解  理解事务之前，先讲一个你日常生活中最常干的事：取钱。  比如你去ATM机取100\",\"icon\":{\"type\":\"icon\",\"url\":\"https://csdnimg.cn/public/favicon.ico\",\"aspectRatio\":0}}" %}

{% embed data="{\"url\":\"https://blog.csdn.net/Java\_3y/article/details/81173287\",\"type\":\"link\",\"title\":\"数据库两大神器【索引和锁】 - CSDN博客\",\"description\":\"前言     只有光头才能变强   索引和锁在数据库中可以说是非常重要的知识点了，在面试中也会经常会被问到的。  本文力求简单讲清每个知识点，希望大家看完能有所收获     声明：如果没有说明具体的数据库和存储引擎，默认指的是MySQL中的InnoDB存储引擎     一、索引  在之前，我对索引有以下的认知：   索引可以加快数据库的检索速度 表经常进行INSERT/UPDATE/DELETE操...\",\"icon\":{\"type\":\"icon\",\"url\":\"https://csdnimg.cn/public/favicon.ico\",\"aspectRatio\":0}}" %}

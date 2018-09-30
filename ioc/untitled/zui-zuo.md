# 2.5 最左前缀匹配原则

## 1. 解释

```sql
CREATE TABLE `user2` (
  `userid` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(20) NOT NULL DEFAULT '',
  `password` varchar(20) NOT NULL DEFAULT '',
  `usertype` varchar(20) NOT NULL DEFAULT '',
  PRIMARY KEY (`userid`),
  KEY `a_b_c_index` (`username`,`password`,`usertype`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
```

 当存在username时会使用索引查询：

```sql
explain select * from user2 where username = '1' and password = '1';
```

![](../../.gitbook/assets/image%20%28170%29.png)

 当没有username时，不会使用索引查询：

```sql
explain select * from user2 where password = '1';
```

![](../../.gitbook/assets/image%20%28215%29.png)

 当有username，但顺序乱序时也可以使用索引：

```sql
explain select * from user2 where password = '1' and username = '1';
```

![](../../.gitbook/assets/image%20%28288%29.png)

观察上述两个explain结果中的type字段。查询中分别是：

* type: index
* type: ref

**index**：这种类型表示mysql会**对整个该索引进行扫描**。要想用到这种类型的索引，对这个索引并无特别要求，只要是索引，或者某个联合索引的一部分，mysql都可能会采用index类型的方式扫描。但是呢，缺点是效率不高，mysql会从索引中的第一个数据一个个的查找到最后一个数据，直到找到**符合判断条件的某个索引**。所以，**上述语句会触发索引**。

  
**ref**：这种类型表示mysql会根据特定的算法**快速查找到某个符合条件的索引**，而不是会对索引中每一个数据都进行一一的扫描判断，也就是所谓你平常理解的使用索引查询会更快的取出数据。而要想实现这种查找，索引却是有要求的，要实现这种能快速查找的算法，索引就要满足特定的数据结构。简单说，也就是**索引字段的数据必须是有序**的，才能实现这种类型的查找，才能利用到索引。

总结：

* index：mysql会对**整个该索引**进行扫描，效率低，会遍历索引中数据知道符合为止，触发：**只要是索引，或者是某个复合索引的一部分**；
* ref：mysql内部使用特定算法优化过的，触发：**索引字段数据必须有序**。

## 2. 注意点

在最左匹配原则中，有如下说明：

1. **最左前缀匹配原则**，非常重要的原则，mysql会一直向右匹配直到遇到范围查询\(&gt;、&lt;、between、like\)就停止匹配，比如a = 1 and b = 2 and c &gt; 3 and d = 4 如果建立\(a,b,c,d\)顺序的索引，d是用不到索引的，如果建立\(a,b,d,c\)的索引则都可以用到，a,b,d的顺序可以任意调整。
2. **=和in可以乱序**，比如a = 1 and b = 2 and c = 3 建立\(a,b,c\)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式




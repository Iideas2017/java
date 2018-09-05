# 1.1 连接

## 一、前提

### **1. 建表语句**

```sql
CREATE TABLE `a_table` (
  `a_id` int(11) DEFAULT NULL,
  `a_name` varchar(10) DEFAULT NULL,
  `a_part` varchar(10) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

```sql
CREATE TABLE `b_table` (
  `b_id` int(11) DEFAULT NULL,
  `b_name` varchar(10) DEFAULT NULL,
  `b_part` varchar(10) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

![&#x8868;&#x5185;&#x5185;&#x5BB9;](../../.gitbook/assets/image%20%2899%29.png)

### 2. 内连接

关键字：`inner join on`

语句：`select * from a_table a inner join b_table b on a.a_id = b.b_id;`

![&#x7ED3;&#x679C;](../../.gitbook/assets/image%20%2845%29.png)

![&#x4E24;&#x4E2A;&#x8868;&#x7684;&#x4EA4;&#x96C6;](../../.gitbook/assets/image%20%28100%29.png)

### 3. 左连接（左外连接）

关键字：`left join on / left outer join on`

语句：`select * from a_table a left join b_table b on a.a_id = b.b_id;`

![&#x67E5;&#x8BE2;&#x7ED3;&#x679C;](../../.gitbook/assets/image%20%2862%29.png)

![&#x5DE6;&#x5916;&#x8FDE;&#x63A5;](../../.gitbook/assets/image%20%282%29.png)

### 4. 右连接（右外连接）

关键字：`right join on / right outer join on`

语句：`select * from a_table a right outer join b_table b on a.a_id = b.b_id;`

![&#x67E5;&#x8BE2;&#x7ED3;&#x679C;](../../.gitbook/assets/image%20%2844%29.png)

![&#x53F3;&#x8FDE;&#x63A5;](../../.gitbook/assets/image%20%2839%29.png)

### 5. 全连接（全外连接）




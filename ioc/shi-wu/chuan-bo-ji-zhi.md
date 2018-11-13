# 3.3 传播机制

## 1. 概述

| 传播行为 | 含义 |
| :--- | :--- |
| PROPAGATION\_REQUIRED | 表示当前方法必须运行在事务中。如果当前事务存在，方法将会在该事务中运行。否则，会启动一个新的事务 |
| PROPAGATION\_SUPPORTS | 表示当前方法不需要事务上下文，但是如果存在当前事务的话，那么该方法会在这个事务中运行 |
| PROPAGATION\_MANDATORY | 表示该方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常 |
| PROPAGATION\_REQUIRED\_NEW | 表示当前方法必须运行在它自己的事务中。一个新的事务将被启动。如果存在当前事务，在该方法执行期间，当前事务会被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager |
| PROPAGATION\_NOT\_SUPPORTED | 表示该方法不应该运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager |
| PROPAGATION\_NEVER | 表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛出异常 |
| PROPAGATION\_NESTED | 表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，那么其行为与PROPAGATION\_REQUIRED一样。注意各厂商对这种传播行为的支持是有所差异的。可以参考资源管理器的文档来确认它们是否支持嵌套事务 |

一般SSH的项目都是使用三层架构即Controller、Services、DAO。  
 Spring 的事务一般都在Services定义，而Controller、DAO都不定义事务。  
 那么 Services 方法调用 Services 的方法，事务是怎么执行的？  
 有些人说不建议Service 调用Service，或者如果要Service 调用Service必须使用嵌套事务。真的是这样的吗？

带着疑问继续了解Spring事务传播行为

spring事务定义了7种传播行为，传播行为有什么作用？在什么情况下使用？

## 2. Spring 事务传播行为

Spring的TransactionDefinition类中定义了7中事务传播类型，代码如下：

![](../../.gitbook/assets/image%20%28114%29.png)

我们先来假设一个场景  
 在 ServiceA 中方法 A\(\) 调用 ServiceB 中方法 B\(\)。  
 Spring 的**事务传播行为**就是解决方法之间的事务传播的。  
 基本方法调用场景如下：

* 方法A有事务，方法B也有事务
* 方法A有事务，方法B没有事务
* 方法A没有事务，方法B有事务
* 方法A没有事务，方法B也没有事务

### **1. REQUIRED**

**支持当前事务**，如果当前没有事务，就新建一个事务。他也是Spring提供的**默认事务**传播行为，适合绝大数情况。

如果A方法有事务，那么B方法就使用A方法的事务。  
 如果A方法没有事务，那么B方法就创建一个新事务。

### **2. SUPPORTS**

支持当前事务，如果当前没有事务，就以非事务方式执行。

如果A方法有事务，那么B方法就使用A方法的事务。  
 如果A方法没有事务，那么B方法就不使用事务的方式执行。

### **3. MANDATORY**

支持当前事务，如果当前没有事务，就抛出异常。

如果A方法有事务，那么A方法就使用A方法事务。  
 如果A方法没有事务，那么就抛出异常。

> 该事务传播行为要求A方法必须以事务的方式运行

### **4. REQUIRES\_NEW**

新建事务，如果当前存在事务，把当前事务挂起。

如果A方法有事务，就把A方法的事务挂起，B方法新创建一个事务。  
如果A方法没有事务，那么B方法就创建一个新事务。

### **5. NOT\_SUPPORTED**

以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。

如果A方法有事务，那么就把A方法的事务挂起，B方法以非事务的方式执行。  
如果A方法没有事务，那么B也不使用事务执行。

### **6. NEVER**

以非事务方式执行，如果当前存在事务，则抛出异常。

如果方法A有事务，那么就抛出异常。  
如果方法A没有事务，那么B方法就以非事务的方式运行。

> 跟 3. PROPAGATION\_MANDATORY 事务传播行为相反。

### **7. NESTED**

如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与PROPAGATION\_REQUIRED类似的。

如果A方法有事务，那么B方法就在A方法的事务中使用嵌套事务。  
如果A方法没有事务，那么方法B就新创建一个事务。

## 3. 嵌套事务

嵌套事务是使用数据库的SavePoint\(事务保存点\)。需要底层数据库的支持。  
 如果在Spring使用嵌套事务，需要满足一下3点。

* 需要数据库支持。  可以使用以下代码来判断数据库是否支持。

```java
Connection.getMetaData().supportsSavepoints();
```

* JDK 1.4 才支持 java.sql.Savepoint 。所以JDK必须在1.4 及以上。
* 还需要Spring中配置nestedTransactionAllowed=true。  我们要设置 transactionManager 的 nestedTransactionAllowed 属性为 true, 注意, 此属性默认为 false!!!

```markup
<bean id="transactionManager"  
        class="org.springframework.orm.hibernate4.HibernateTransactionManager">  
        <property name="sessionFactory" ref="sessionFactory" />  
        <property name="nestedTransactionAllowed">  
            <value>true</value>  
        </property>  
    </bean> 
```

## 4. 只读事务（Readonly Transaction）

Spring 为了忽略那些不需要事务的方法，比如读取数据，这样可以有效的提高一些性能。

事务的第三个特性是它是否为只读事务。如果事务只对后端的数据库进行该操作，数据库可以利用事务的只读特性来进行一些特定的优化。通过将事务设置为只读，你就可以给数据库一个机会，让它应用它认为合适的优化措施。

## 5. 事务超时 （Transaction Timeout）

为了解决事务执行时间太长，消耗太多资源的问题，可以设置一个超时时间。如果该事务支持超过设置的时间，就回滚该事务。

为了使应用程序很好地运行，事务不能运行太长的时间。因为事务可能涉及对后端数据库的锁定，所以长时间的事务会不必要的占用数据库资源。事务超时就是事务的一个定时器，在特定时间内事务如果没有执行完毕，那么就会自动回滚，而不是一直等待其结束。

## 6. 回滚规则

事务五边形的最后一个方面是一组规则，这些规则定义了哪些异常会导致事务回滚而哪些不会。默认情况下，事务只有遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚（这一行为与EJB的回滚行为是一致的）   
但是你可以声明事务在遇到特定的检查型异常时像遇到运行期异常那样回滚。同样，你还可以声明事务遇到特定的异常不回滚，即使这些异常是运行期异常。

## 


# 1. Spring 框架概述

## 1. 概述

Spring被设计为非侵入式的，这意味着你的域逻辑代码通常不会依赖于框架本身。在集成层（比如数据访问层），会存在一些依赖同时依赖于数据访问技术和Spring，但是这些依赖可以很容易地从代码库中分离出来。

Spring框架是基于Java平台的，它为开发Java应用提供了全方位的基础设施支持，并且它很好地处理了这些基础设施，所以你只需要关注你的应用本身即可。

Spring可以使用POJO（普通的Java对象，plain old java objects）创建应用，并且可以将企业服务非侵入式地应用到POJO。这项功能适用于Java SE编程模型以及全部或部分的Java EE。

不用关心事务API就可以执行数据库事务；

不用关心远程API就可以使用远程操作；

不用关心JMX API就可以进行管理操作；

不用关心JMS API就可以进行消息处理。

①JMX，Java Management eXtension，Java管理扩展，是一个为应用程序、设备、系统等植入管理功能的框架。JMX可以跨越一系列异构操作系统平台、系统体系结构和网络传输协议，灵活的开发无缝集成的系统、网络和服务管理应用。

②JMS，Java Message Service，Java消息服务，是Java平台上有关面向消息中间件\(MOM\)的技术规范，它便于消息系统中的Java应用程序进行消息交换,并且通过提供标准的产生、发送、接收消息的接口简化企业应用的开发。

## 2. 框架特征与功能

**轻量：**从大小与开销两方面而言Spring都是轻量的。完整的Spring框架可以在一个大小只有1MB多的JAR文件里发布。并且Spring所需的处理开销也是微不足道的。此外，Spring是非侵入式的：典型地，Spring应用中的对象不依赖于Spring的特定类。

**控制反转Ioc：**Spring通过一种称作控制反转（IoC）的技术促进了低耦合。当应用了IoC，一个对象依赖的其它对象会通过被动的方式传递进来，而不是这个对象自己创建或者查找依赖对象。你可以认为IoC与JNDI相反——不是对象从容器中查找依赖，而是容器在对象初始化时不等对象请求就主动将依赖传递给它。

**面向切面Aop：**Spring提供了面向切面编程的丰富支持，允许通过分离应用的业务逻辑与系统级服务（例如审计（auditing）和事务（transaction）管理）进行**内聚性**的开发。应用对象只实现它们应该做的——完成业务逻辑——仅此而已。它们并不负责（甚至是意识）其它的系统级关注点，例如**日志或事务**支持。

![Spring FrameWork](../../.gitbook/assets/image%20%28351%29.png)

**容器：**Spring包含并管理应用对象的配置和生命周期，在这个意义上它是一种容器，你可以配置你的每个bean如何被创建——基于一个可配置原型（prototype），你的bean可以创建一个单独的实例或者每次需要时都生成一个新的实例——以及它们是如何相互关联的。然而，Spring不应该被混同于传统的重量级的EJB容器，它们经常是庞大与笨重的，难以使用。

**框架：**Spring可以将简单的组件配置、组合成为复杂的应用。在Spring中，应用对象被声明式地组合，典型地是在一个XML文件里。Spring也提供了很多基础功能（事务管理、持久化框架集成等等），将应用逻辑的开发留给了你。

**MVC：**Spring的作用是整合，但不仅仅限于整合，Spring 框架可以被看做是一个企业解决方案级别的框架，Spring MVC是一个非常受欢迎的轻量级Web框架。

## 3. Spring组成

Spring 框架是一个分层架构，由 7 个定义良好的模块组成。Spring 模块构建在核心容器之上，核心容器定义了创建、配置和管理 bean 的方式。

![Spring&#x7EC4;&#x6210;](../../.gitbook/assets/image%20%28248%29.png)

组成 Spring 框架的每个模块（或组件）都可以单独存在，或者与其他一个或多个模块联合实现。每个模块的功能如下：

* **核心容器**：核心容器提供 Spring 框架的基本功能。核心容器的主要组件是 BeanFactory，它是工厂模式的实现。**BeanFactory** 使用_控制反转_（IOC） 模式将应用程序的配置和依赖性规范与实际的应用程序代码分开。
* **Spring 上下文**：Spring 上下文是一个配置文件，向 Spring 框架提供上下文信息。Spring 上下文包括企业服务，例如 JNDI、EJB、电子邮件、国际化、校验和调度功能。
* **Spring AOP**：通过配置管理特性，Spring AOP 模块直接将面向方面的编程功能集成到了 Spring 框架中。所以，可以很容易地使 Spring 框架管理的任何对象支持 AOP。Spring AOP 模块为基于 Spring 的应用程序中的对象提供了事务管理服务。通过使用 Spring AOP，不用依赖 EJB 组件，就可以将声明性事务管理集成到应用程序中。
* **Spring DAO**：JDBC DAO 抽象层提供了有意义的异常层次结构，可用该结构来管理异常处理和不同数据库供应商抛出的错误消息。异常层次结构简化了错误处理，并且极大地降低了需要编写的异常代码数量（例如打开和关闭连接）。Spring DAO 的面向 JDBC 的异常遵从通用的 DAO 异常层次结构。
* **Spring ORM**：Spring 框架插入了若干个 ORM 框架，从而提供了 ORM 的对象关系工具，其中包括 JDO、Hibernate 和 iBatis SQL Map。所有这些都遵从 Spring 的通用事务和 DAO 异常层次结构。
* **Spring Web 模块**：Web 上下文模块建立在应用程序上下文模块之上，为基于 Web 的应用程序提供了上下文。所以，Spring 框架支持与 Jakarta Struts 的集成。Web 模块还简化了处理多部分请求以及将请求参数绑定到域对象的工作。
* **Spring MVC 框架**：MVC 框架是一个全功能的构建 Web 应用程序的 MVC 实现。通过策略接口，MVC 框架变成为高度可配置的，MVC 容纳了大量视图技术，其中包括 JSP、Velocity、Tiles、iText 和 POI。

## 4. Jar 包介绍

![](../../.gitbook/assets/image%20%28190%29.png)

 core部分包含5个模块

1. spring-core：依赖注入IoC与DI的最基本实现
2. spring-beans：Bean工厂与bean的装配
3. spring-context：spring的context上下文即**IoC容器**
4. spring-context-support 
5. spring-expression：spring表达式语言

![](../../.gitbook/assets/image%20%28290%29.png)


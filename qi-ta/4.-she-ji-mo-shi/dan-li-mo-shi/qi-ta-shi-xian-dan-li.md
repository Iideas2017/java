# 6. 其他实现单例

## 使用static代码块实现单例

静态代码块中的代码在使用类的时候就已经执行了，所以可以应用静态代码块的这个特性的实现单例设计模式。

```java
package org.mlinge.s08;
 
public class MySingleton{
	 
	private static MySingleton instance = null;
	 
	private MySingleton(){}
 
	static{
		instance = new MySingleton();
	}
	
	public static MySingleton getInstance() { 
		return instance;
	} 
}

```

## 使用枚举数据类型实现单例模式

枚举enum和静态代码块的特性相似，在使用枚举时，构造方法会被自动调用，利用这一特性也可以实现单例：

```java
package org.mlinge.s09;
 
public enum EnumFactory{ 
    
    singletonFactory;
    
    private MySingleton instance;
    
    private EnumFactory(){//枚举类的构造方法在类加载是被实例化
        instance = new MySingleton();
    }
        
    public MySingleton getInstance(){
        return instance;
    }
    
}
 
class MySingleton{//需要获实现单例的类，比如数据库连接Connection
    public MySingleton(){} 
}

```

 运行结果表明单例得到了保证，但是这样写枚举类被完全暴露了，据说违**反了“职责单一原则”**，那我们来看看怎么进行改造呢。

### **完善使用enum枚举实现单例**

```java
package org.mlinge.s10;
public class ClassFactory{
private enum MyEnumSingleton{
    singletonFactory;

    private MySingleton instance;

    private MyEnumSingleton(){//枚举类的构造方法在类加载是被实例化
        instance = new MySingleton();
    }

    public MySingleton getInstance(){
        return instance;
    }
} 

public static MySingleton getInstance(){
    return MyEnumSingleton.singletonFactory.getInstance();
}
}

class MySingleton{
//需要获实现单例的类，比如数据库连接Connection 

public MySingleton(){} 

}
```








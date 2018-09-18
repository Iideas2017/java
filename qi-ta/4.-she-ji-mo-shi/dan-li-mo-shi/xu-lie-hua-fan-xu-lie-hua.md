# 4. 序列化反序列化

          静态内部类虽然保证了单例在多线程并发下的线程安全性，但是在遇到序列化对象时，默认的方式运行得到的结果就是多例的。

代码实现如下：

```java
package org.mlinge.s07;
 
import java.io.Serializable;
 
public class MySingleton implements Serializable {
	 
	private static final long serialVersionUID = 1L;
 
	//内部类
	private static class MySingletonHandler{
		private static MySingleton instance = new MySingleton();
	} 
	
	private MySingleton(){}
	 
	public static MySingleton getInstance() { 
		return MySingletonHandler.instance;
	}
}

```

 序列化与反序列化测试代码：

```java
package org.mlinge.s07;
 
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
 
public class SaveAndReadForSingleton {
	
	public static void main(String[] args) {
		MySingleton singleton = MySingleton.getInstance();
		
		File file = new File("MySingleton.txt");
		
		try {
			FileOutputStream fos = new FileOutputStream(file);
			ObjectOutputStream oos = new ObjectOutputStream(fos);
			oos.writeObject(singleton);
			fos.close();
			oos.close();
			System.out.println(singleton.hashCode());
		} catch (FileNotFoundException e) { 
			e.printStackTrace();
		} catch (IOException e) { 
			e.printStackTrace();
		}
		
		try {
			FileInputStream fis = new FileInputStream(file);
			ObjectInputStream ois = new ObjectInputStream(fis);
			MySingleton rSingleton = (MySingleton) ois.readObject();
			fis.close();
			ois.close();
			System.out.println(rSingleton.hashCode());
		} catch (FileNotFoundException e) { 
			e.printStackTrace();
		} catch (IOException e) { 
			e.printStackTrace();
		} catch (ClassNotFoundException e) { 
			e.printStackTrace();
		}
		
	}
}

```

 运行以上代码，得到的结果如下：

```text
865113938
1442407170
```

从结果中我们发现，序列号对象的hashCode和反序列化后得到的对象的hashCode值不一样，说明反序列化后返回的对象是重新实例化的，单例被破坏了。那怎么来解决这一问题呢？

解决办法就是在反序列化的过程中**使用readResolve\(\)方法**，单例实现的代码如下：

```java
package org.mlinge.s07;
 
import java.io.ObjectStreamException;
import java.io.Serializable;
 
public class MySingleton implements Serializable {
	 
	private static final long serialVersionUID = 1L;
 
	//内部类
	private static class MySingletonHandler{
		private static MySingleton instance = new MySingleton();
	} 
	
	private MySingleton(){}
	 
	public static MySingleton getInstance() { 
		return MySingletonHandler.instance;
	}
	
	//该方法在反序列化时会被调用，该方法不是接口定义的方法，有点儿约定俗成的感觉
	protected Object readResolve() throws ObjectStreamException {
		System.out.println("调用了readResolve方法！");
		return MySingletonHandler.instance; 
	}
}
 

```

 再次运行上面的测试代码，得到的结果如下：

```text
865113938
调用了readResolve方法！
865113938
```

从运行结果可知，添加readResolve方法后反序列化后得到的实例和序列化前的是同一个实例，单个实例得到了保证。


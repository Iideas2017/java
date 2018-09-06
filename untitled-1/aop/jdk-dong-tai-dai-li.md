# Jdk 动态代理

## 1. 示例

### **1.1 需要代理的接口**

```text
public interface IHello {
    public void sayHello();
}
```

### **1.2 需要代理的类**

```text
public class HelloImpl implements IHello {
    public void sayHello() {
        System.out.println("Hello World...");
    }
}
```

### **1.3 调用处理器实现类**

```text
public class ProxyHandler implements InvocationHandler {
    private Object target;
    public ProxyHandler(Object target) {
        this.target = target;
    }
    public Object proxyInstance() {
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),
                target.getClass().getInterfaces(), this);
    }
    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
        System.out.println("aspect before ... ");
        Object result = method.invoke(this.target, args);
        System.out.println("aspect after ... ");
        return result;
    }
}
```

### **1.4 测试类入口**

```text
public class Main {
    public static void main(String[] args) {
        ProxyHandler proxy = new ProxyHandler(new HelloImpl());
        IHello hello = (IHello) proxy.proxyInstance();
        hello.sayHello();
    }
}
```

  
  



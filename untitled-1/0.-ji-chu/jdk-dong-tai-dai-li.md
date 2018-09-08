# 3. Jdk 动态代理

## 1. AOP

  在java的动态代理机制中，有两个重要的类或接口，一个是 **InvocationHandler**\(Interface\)、 另一个则是 **Proxy**\(Class\)。

### 1.1 **InvocationHandler 接口**

每一个动态代理类都**必须**要**实现InvocationHandler这个接口**，并且每个代理类的实例都关联到了一个handler，当我们通过代理对象调用一个方法的时候，这个方法的调用就会被转发为由InvocationHandler这个接口的 invoke 方法来进行调用。我们来看看InvocationHandler这个接口的唯一一个方法 invoke 方法：

```java
Object invoke(Object proxy, Method method, Object[] args) throws Throwable

proxy:　　 指代我们所代理的那个真实对象
method:　　指代的是我们所要调用真实对象的某个方法的Method对象
args:　　  指代的是调用真实对象某个方法时接受的参数
```

### 1.2 Proxy 类

Proxy这个类的作用就是用来动态创建一个代理对象的类，它提供了许多的方法，但是我们用的最多的就是 newProxyInstance 这个方法。

这个方法的作用就是得到一个动态的代理对象，其接收三个参数，我们来看看这三个参数所代表的含义：

```java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) throws IllegalArgumentException

loader:　　一个ClassLoader对象，定义了由哪个ClassLoader对象来对生成的代理对象进行加载

interfaces:　　一个Interface对象的数组，表示的是我将要给我需要代理的对象提供一组什么接口，如果我提供了一组接口给它，那么这个代理对象就宣称实现了该接口(多态)，这样我就能调用这组接口中的方法了

h:　　一个InvocationHandler对象，表示的是当我这个动态代理对象在调用方法的时候，会关联到哪一个InvocationHandler对象
```

## 2. 示例

首先我们定义了一个Subject类型的接口，为其声明了两个方法：

```java
public interface Subject
{
    public void rent();
    
    public void hello(String str);
}
```

接着，定义了一个类来实现这个接口，这个类就是我们的真实对象，RealSubject类：

```java
public class RealSubject implements Subject
{
    @Override
    public void rent()
    {
        System.out.println("I want to rent my house");
    }
    
    @Override
    public void hello(String str)
    {
        System.out.println("hello: " + str);
    }
```

下一步，我们就要定义一个**动态代理类**了，前面说个，每一个动态代理类都必须要实现 InvocationHandler 这个接口，因此我们这个动态代理类也不例外：

```java
public class DynamicProxy implements InvocationHandler
{
    // 这个就是我们要代理的真实对象
    private Object subject;
    
    // 构造方法，给我们要代理的真实对象赋初值
    public DynamicProxy(Object subject)
    {
        this.subject = subject;
    }
    
    @Override
    public Object invoke(Object object, Method method, Object[] args)
            throws Throwable
    {
        //　在代理真实对象前我们可以添加一些自己的操作
        System.out.println("before rent house");
        System.out.println("Method:" + method);
        
        //当代理对象调用真实对象的方法时，其会自动的跳转到代理对象
        //关联的handler对象的invoke方法来进行调用
        method.invoke(subject, args);
        
        //在代理真实对象后我们也可以添加一些自己的操作
        System.out.println("after rent house");
        
        return null;
    }
```

最后，来看看我们的Main 方法：

```java
public class Client
{
    public static void main(String[] args)
    {
        //我们要代理的真实对象
        Subject realSubject = new RealSubject();
        //我们要代理哪个真实对象，就将该对象传进去，
        //最后是通过该真实对象来调用其方法的
        InvocationHandler handler = new DynamicProxy(realSubject);

        /*
         * 通过Proxy的newProxyInstance方法来创建我们的代理对象，
           我们来看看其三个参数
           
         * 第一个参数 handler.getClass().getClassLoader() ，
           我们这里使用handler这个类的ClassLoader对象来加载我们的代理对象
           
         * 第二个参数realSubject.getClass().getInterfaces()，
           我们这里为代理对象提供的接口是真实对象所实行的接口，
           表示我要代理的是该真实对象，这样我就能调用这组接口中的方法了
           
         * 第三个参数handler， 我们这里将这个代理对象关联到了上方的 
           InvocationHandler 这个对象上
         */
         
        Subject subject = (Subject)Proxy.newProxyInstance(
        handler.getClass().getClassLoader(), 
        realSubject.getClass().getInterfaces(),
        handler);
        
        System.out.println(subject.getClass().getName());
        subject.rent();
        subject.hello("world");
    }
}
```

### 流程

1. 定义接口；
2. 定义接口实现类；
3. 定义动态代理类，如何定义？
   1. 实现 InvocationHandler 接口；
   2. 重写 invoke 方法
   3. 
4. 定义main方法
   1. 定义实现类对象，作为参数传入代理类对象，生成代理类对象；
   2. 代理类对象的ClassLoader，实现类对象实现的接口，代理类对象作为参数，传入Proxy.newProxyInstance（）参数里面，生成接口对象；
   3. 调用接口对象的方法。

        **同时我们一定要记住，通过 Proxy.newProxyInstance 创建的代理对象是在jvm运行时动态生成的一个对象，它并不是我们的InvocationHandler类型，也不是我们定义的那组接口的类型，而是在运行是动态生成的一个对象，并且命名方式都是这样的形式，以$开头，proxy为中，最后一个数字表示对象的标号**。

接着我们来看看这两句 

subject.rent\(\);  
subject.hello\("world"\);

这里是通过**代理对象**来调用实现的那种接口中的方法，这个时候程序就会跳转到由这个代理对象关联到的 handler 中的invoke方法去执行，而我们的这个 handler 对象又接受了一个 RealSubject类型的参数，表示我要代理的就是这个真实对象，所以此时就会调用 handler 中的invoke方法去执行


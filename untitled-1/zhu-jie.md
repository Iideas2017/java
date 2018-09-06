# 4. 注解

## 1. 注解定义

 注解就是元数据，即一种描述数据的数据。所以，可以说注解就是源代码的元数据。

```java
@Override
public String toString() {
return"This is String Representation of current object.";
}
```

事实上，@Override告诉编译器这个方法是一个重写方法\(描述方法的元数据\)，如果父类中不存在该方法，编译器便会报错，提示该方法没有重写父类中的方法。如果我不小心拼写错误，例如将toString\(\)写成了toStrring\(\){double r}，而且我也没有使用@Override注解，那程序依然能编译运行。但运行结果会和我期望的大不相同。现在我们了解了什么是注解，并且使用注解有助于阅读程序。

Annotation是一种应用于类、方法、参数、变量、构造器及包声明中的特殊修饰符。它是一种由JSR-175标准选择用来描述元数据的一种工具。

## 2. 常见标准的Annotation

###  1）Override

         java.lang.Override是一个**标记类型**注解，它被用作**标注方法**。它说明了被标注的方法重载了父类的方法，**起到了断言**的作用。如果我们使用了这种注解在一个没有覆盖父类方法的方法时，java编译器将以一个编译错误来警示。

### 2）Deprecated

        Deprecated也是一种**标记类型**注解。当一个**类型或者类型成员**使用@Deprecated修饰的话，编译器将**不鼓励使用这个被标注的程序元素。**所以使用这种修饰具有一定的“延续性”：如果我们在代码中通过继承或者覆盖的方式使用了这个过时的类型或者成员，虽然继承或者覆盖后的类型或者成员并不是被声明为@Deprecated，但编译器仍然要报警。

### 3）SuppressWarnings

        SuppressWarning不是一个标记类型注解。它**有一个类型为String\[\]的成员**，这个成员的值为被禁止的警告名。对于javac编译器来讲，被-Xlint选项有效的警告名也同样对@SuppressWarings有效，同时编译器忽略掉无法识别的警告名。  
　　@SuppressWarnings\("unchecked"\)

## 3. 为什么要引入注解？

 使用Annotation之前\(甚至在使用之后\)，XML被广泛的应用于**描述元数据**。不知何时开始一些应用开发人员和架构师发现XML的维护越来越糟糕了。他们希望使用一些和代码紧耦合的东西，而不是像XML那样和代码是松耦合的\(在某些情况下甚至是完全分离的\)代码描述。

假如你想为应用设置很多的**常量或参数**，这种情况下，XML是一个很好的选择，因为它不会同特定的代码相连; 如果你想把某个方法声明为**服务**，那么**使用Annotation**会更好一些，因为这种情况下需要注解和方法紧密耦合起来，开发人员也必须认识到这点。

另一个很重要的因素是Annotation定义了一种**标准的描述元数据的方式**。在这之前，开发人员通常使用他们自己的方式定义元数据。例如，使用标记interfaces，注释，transient关键字等等。每个程序员按照自己的方式定义元数据，而不像Annotation这种标准的方式。

目前，许多框架将XML和Annotation两种方式结合使用，平衡两者之间的利弊。

## 4.  元注解

J2SE5.0版本在 java.lang.annotation提供了四种元注解，专门注解其他的注解：

`@Documented –注解是否将包含在JavaDoc中  
@Retention –什么时候使用该注解  
@Target –注解用于什么地方  
@Inherited – 是否允许子类继承该注解`

**@Documented**–一个简单的Annotations标记注解，表示是否将注解信息添加在java文档中。

**@Retention**– 定义该注解的生命周期 \(限定注解的可见范围\)

1. SOURCE —— 这种类型的Annotations**只在源代码级别保留**,编译时就会被忽略   
2. CLASS —— 这种类型的Annotations**编译时**被保留,在class文件中存在,但JVM将会忽略   
3. RUNTIME —— 这种类型的Annotations将**被JVM保留**,所以他们能在运行时**被JVM或其他使用反射机制的代码所读取和使用**. 

**RetentionPolicy.SOURCE** – 在编译阶段丢弃。这些注解在编译结束之后就不再有任何意义，所以它们不会写入字节码。@Override, @SuppressWarnings都属于这类注解。

**RetentionPolicy.CLASS** – 在类加载的时候丢弃。在字节码文件的处理中有用。注解默认使用这种方式。

**RetentionPolicy.RUNTIME**– 始终不会丢弃，运行期也保留该注解，因此可以使用反射机制读取该注解的信息。我们自定义的注解通常使用这种方式。

![](../.gitbook/assets/image%20%28124%29.png)

![&#x53EF;&#x89C1;&#x8303;&#x56F4;](../.gitbook/assets/image%20%28104%29.png)

**@Target** – 表示该注解用于什么地方。如果不明确指出，该注解可以放在任何地方。以下是一些可用的参数。需要说明的是：属性的注解是兼容的，如果你想给7个属性都添加注解，仅仅排除一个属性，那么你需要在定义target包含所有的属性。

`ElementType.TYPE:用于描述类、接口或enum声明  
ElementType.FIELD:用于描述实例变量  
ElementType.METHOD 用于描述方法  
ElementType.PARAMETER 用于描述参数  
ElementType.CONSTRUCTOR 用于描述构造器  
ElementType.LOCAL_VARIABLE 用于描述局部变量  
ElementType.ANNOTATION_TYPE 另一个注释  
ElementType.PACKAGE 用于记录java文件的package信息`

![](../.gitbook/assets/image%20%2822%29.png)



**@Inherited** – 定义该注释和子类的关系

## 5. Annotation是如何工作的？

  大体分为三部分: 定义注解、使用注解、解析注解

 自定义注解类编写的一些规则:  
  1. Annotation型定义为**@interface**, 所有的Annotation会**自动继承**java.lang.Annotation这一接口,并且不能再去继承别的类或是接口.  
  2. 参数成员**只能用public或默认\(default\)**这两个访问权修饰  
  3. 参数成员只能用**基本类型**byte,short,char,int,long,float,double,boolean八种基本数据类型和String、Enum、Class、annotations**等数据类型,**以及这一些类型的数组.  
  4. 要获取**类方法和字段的注解信息**，必须通过Java的**反射技术**来获取 Annotation对象,因为你除此之外没有别的获取注解对象的方法  
  5. 注解也可以没有定义成员, 不过这样注解就没啥用了

### 1\) 定义注解:

```java
  import java.lang.annotation.ElementType;
  import java.lang.annotation.Retention;
  import java.lang.annotation.RetentionPolicy;
  import java.lang.annotation.Target;
  
  
  @Target(ElementType.METHOD)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface MyAnnotation {
     //定义注解的属性，这不是方法
     String name();//必选注解
     int value() default 20;//有属性就是可选属性

```

###   2\) 注解的使用:

```java
package annotation;

public class UseMyAnnotion {

//    这里只用一个属性，另一个value属性有默认值不用设置
    @MyAnnotation(name = "QizoZhi")
    public void show(String str){
        System.out.println(str);
    }
}
```

###  3\) 解析注解:

        这里使用了底层的映射原理

```java
package annotation;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class MyAnnotationParser {

    public static void main(String[] args) throws NoSuchMethodException, SecurityException, IllegalAccessException, IllegalArgumentException, InvocationTargetException {
//        获取字节码对象
        Class clazz = UseMyAnnotion.class;
        Method method = clazz.getMethod("show", String.class);
//        获取方法上面的注解
        MyAnnotation annotation = method.getAnnotation(MyAnnotation.class);
//        获取注解属性值
        System.out.println(annotation.name()+"\t"+annotation.value());
        
//        取到值就可以根据业务处理数据
        
        //激活方法，也就是让方法执行
        method.invoke(new UseMyAnnotion(), "HH");
    }
}
```


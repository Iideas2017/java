# 2.7 String 与 \(深\)克隆

## 1、克隆的定义与意义

　　顾名思义，克隆就是**制造一个对象的副本**。一般地，根据所要克隆的**对象的成员变量中是否含有引用类型**，可以将克隆分为两种：**浅克隆\(Shallow Clone\) 和 深克隆\(Deep Clone\)**，默认情况下使用Object中的**clone方法进行克隆就是浅克隆**，即完成对象域对域的拷贝。

### **\(1\). Object 中的 clone\(\) 方法**

![](../../.gitbook/assets/image%20%28164%29.png)

        在使用clone\(\)方法时，若该类未实现 Cloneable 接口，则抛出 java.lang.CloneNotSupportedException 异常。下面我们以Employee这个例子进行说明：

```java
public class Employee {
    private String name;
    private double salary;
    private Date hireDay;

    ...
    public static void main(String[] args) throws CloneNotSupportedException {
        Employee employee = new Employee();
        employee.clone();
        System.out.println("克隆完成...");
    } 
}/* Output: 
        ~Exception in thread "main" java.lang.CloneNotSupportedException: P1_1.Employee
 *///:
```

### **\(2\). Cloneable 接口**

　　**Cloneable 接口是一个标识性接口，即该接口不包含任何方法（甚至没有clone\(\)方法），但是如果一个类想合法的进行克隆，那么就必须实现这个接口。**下面我们看JDK对它的描述：

* A class implements the Cloneable interface to indicate to the java.lang.Object.clone\(\) method that it is **legal** for that method to make a field-for-field copy of instances of that class.
* **Invoking Object’s clone method on an instance that does not implement the Cloneable interface results in the exception CloneNotSupportedException being thrown.**
* By convention, classes that implement this interface should **override** Object.clone \(which is protected\) with **a public method**.
* **Note that this interface does not contain the clone\(\) method.** Therefore, it is not possible to clone an object merely by virtue of the fact that it implements this interface. Even if the clone method is invoked reflectively, there is no guarantee that it will succeed.

```java
/**
 * @author  unascribed
 * @see     java.lang.CloneNotSupportedException
 * @see     java.lang.Object#clone()
 * @since   JDK1.0
 */
public interface Cloneable {
}
```

## 2、Clone & Copy

### 1. Copy

     假设现在有一个Employee对象，Employee tobby = new Employee\(“CMTobby”,5000\)，**通常, 我们会有这样的赋值Employee tom=tobby，这个时候只是简单了copy了一下reference**，tom 和 tobby 都指向内存中同一个object，这样tom或者tobby对对象的修改都会影响到对方。打个比方，如果我们通过tom.raiseSalary\(\)方法改变了salary域的值，那么tobby通过getSalary\(\)方法得到的就是修改之后的salary域的值，显然这不是我们愿意看到的。

### 2. clone

     如果我们希望得到tobby所指向的对象的一个精确拷贝，同时两者互不影响，那么我们就可以使用Clone来满足我们的需求。Employee cindy=tobby.clone\(\)，这时会生成一个新的Employee对象，并且和tobby具有相同的属性值和方法。

## 3、Shallow Clone & Deep Clone

　　Clone是如何完成的呢？**Object中的clone\(\)方法在对某个对象实施克隆时对其是一无所知的，它仅仅是简单地执行域对域的copy，这就是Shallow Clone**。这样，问题就来了，以Employee为例，它里面有一个域hireDay不是基本类型的变量，而是一个reference变量，经过Clone之后克隆类只会产生一个新的Date类型的引用，它和原始引用都指向同一个 Date 对象，这样克隆类就和原始类**共享了一部分信息**，显然这种情况不是我们愿意看到的，过程下图所示：

![clone.png-21.7kB](http://static.zybuluo.com/Rico123/xyx4cadhj3ur4myosl7xgopd/clone.png)

　　这个时候，我们就需要进行 Deep Clone 了，以便对那些引用类型的域进行特殊的处理，例如本例中的hireDay。我们可以重新定义 clone方法，对hireDay做特殊处理，如下代码所示：

```java
class Employee implements Cloneable  
{  
    private String name;
    private int id;
    private Date hireDay;
    ...

    @Override
    public Object clone() throws CloneNotSupportedException {
       Employee cloned = (Employee) super.clone();  
       // Date 支持克隆且重写了clone()方法，Date 的定义是：
       // public class Date implements java.io.Serializable, Cloneable, Comparable<Date>
       cloned.hireDay = (Date) hireDay.clone() ;   
       return cloned;  
    }
}  
```

　　因此，Object 在对某个对象实施 Clone 时，对其是一无所知的，**它仅仅是简单执行域对域的Copy**。 其中，**对八种基本类型的克隆是没有问题的，但当对一个引用类型进行克隆时，只是克隆了它的引用。**

       因此，克隆对象和原始对象共享了同一个对象成员变量，故而提出了深克隆 ： 在对整个对象浅克隆后，还需**对其引用变量进行克隆**，并将其更新到浅克隆对象中去。

## 4、一个克隆的示例

　　在这里，我们通过一个简单的例子来说明克隆在Java中的使用，如下所示：

```java
// 父类 Employee 
public class Employee implements Cloneable{

    private String name;
    private double salary;
    private Date hireDay;

    public Employee(String name, double salary, Date hireDay) {
        this.name = name;
        this.salary = salary;
        this.hireDay = hireDay;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public double getSalary() {
        return salary;
    }

    public void setSalary(double salary) {
        this.salary = salary;
    }

    public Date getHireDay() {
        return hireDay;
    }

    public void setHireDay(Date hireDay) {
        this.hireDay = hireDay;
    }

    @Override
    public Object clone() throws CloneNotSupportedException {
        Employee cloned = (Employee) super.clone();
        cloned.hireDay = (Date) hireDay.clone();
        return cloned;
    }


    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result + ((hireDay == null) ? 0 : hireDay.hashCode());
        result = prime * result + ((name == null) ? 0 : name.hashCode());
        long temp;
        temp = Double.doubleToLongBits(salary);
        result = prime * result + (int) (temp ^ (temp >>> 32));
        return result;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj)
            return true;
        if (obj == null)
            return false;
        if (getClass() != obj.getClass())
            return false;
        Employee other = (Employee) obj;
        if (hireDay == null) {
            if (other.hireDay != null)
                return false;
        } else if (!hireDay.equals(other.hireDay))
            return false;
        if (name == null) {
            if (other.name != null)
                return false;
        } else if (!name.equals(other.name))
            return false;
        if (Double.doubleToLongBits(salary) != Double
                .doubleToLongBits(other.salary))
            return false;
        return true;
    }

    @Override
    public String toString() {
        return name + " : " + String.valueOf(salary) + " : " + hireDay.toString();
    }
}
```

```java
// 子类 Manger 
public class Manger extends Employee implements Cloneable {
    private String edu;

    public Manger(String name, double salary, Date hireDay, String edu) {
        super(name, salary, hireDay);
        this.edu = edu;
    }

    public String getEdu() {
        return edu;
    }

    public void setEdu(String edu) {
        this.edu = edu;
    }

    @Override
    public String toString() {
        return this.getName() + " : " + this.getSalary() + " : "
                + this.getHireDay() + " : " + this.getEdu();
    }

    @Override
    public int hashCode() {
        final int prime = 31;
        int result = super.hashCode();
        result = prime * result + ((edu == null) ? 0 : edu.hashCode());
        return result;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj)
            return true;
        if (!super.equals(obj))
            return false;
        if (getClass() != obj.getClass())
            return false;
        Manger other = (Manger) obj;
        if (edu == null) {
            if (other.edu != null)
                return false;
        } else if (!edu.equals(other.edu))
            return false;
        return true;
    }

    public static void main(String[] args) throws CloneNotSupportedException {

        Manger manger = new Manger("Rico", 20000.0, new Date(), "NEU");
        // 输出manger
        System.out.println("Manger对象 = " + manger.toString());

        Manger clonedManger = (Manger) manger.clone();
        // 输出克隆的manger
        System.out.println("Manger对象的克隆对象 = " + clonedManger.toString());
        System.out.println("Manger对象和其克隆对象是否相等：  "
                + manger.equals(clonedManger) + "\r\n");

        // 修改、输出manger
        manger.setEdu("TJU");
        System.out.println("修改后的Manger对象 = " + manger.toString());

        // 再次输出manger
        System.out.println("原克隆对象= " + clonedManger.toString());
        System.out.println("修改后的Manger对象和原克隆对象是否相等：  "
                + manger.equals(clonedManger));
    }
}
/* Output: 
        Manger对象 = Rico : 20000.0 : Mon Mar 13 15:36:03 CST 2017 : NEU
        Manger对象的克隆对象 = Rico : 20000.0 : Mon Mar 13 15:36:03 CST 2017 : NEU
        Manger对象和其克隆对象是否相等：  true

        修改后的Manger对象 = Rico : 20000.0 : Mon Mar 13 15:36:03 CST 2017 : TJU
        原克隆对象= Rico : 20000.0 : Mon Mar 13 15:36:03 CST 2017 : NEU
        修改后的Manger对象和原克隆对象是否相等：  false
 *///:
```

## 5、Clone\(\)方法的保护机制

　　   **在Object中clone\(\)是被申明为 protected** 的，这样做是有一定的道理的。以 Employee 类为例，如果我们在Employee中重写了protected Object clone\(\)方法， 就大大限制了可以“克隆”Employee对象的范围，即可以保证只有在和Employee类在同一包中类及Employee类的子类里面才能“克隆”Employee对象。进一步地，如果我们没有在Employee类重写clone\(\)方法，则只有Employee类及其子类才能够“克隆”Employee对象。

　　这里面涉及到一个大家可能都会忽略的一个知识点，那就是关于protected的用法。实际上，很多的有关介绍Java语言的书籍，都对protected介绍的比较的简单，就是：被protected修饰的成员或方法对于本包和其子类可见。这种说法有点太过含糊，常常会对大家造成误解。

## 6、注意事项

　　Clone\(\)方法的使用比较简单，注意如下几点即可：

1. 什么时候使用shallow Clone，什么时候使用deep Clone？

   　　这个主要看具体对象的域是什么性质的，基本类型还是引用类型。

2. 调用Clone\(\)方法的对象所属的类\(Class\)必须实现 Clonable 接口，否则在调用Clone方法的时候会抛出CloneNotSupportedException；
3. 所有数组对象都实现了 Clonable 接口，默认支持克隆；
4. **如果我们实现了 Clonable 接口，但没有重写Object类的clone方法，那么执行域对域的拷贝；**
5. **明白 String 在克隆中的特殊性**

   　　**String 在克隆时只是克隆了它的引用。**奇怪的是，在修改克隆后的 String 对象时，其原来的对象并未改变。原因是**String是在内存中不可以被改变的对象**。虽然在克隆时，源对象和克隆对象都指向了同一个String对象，但当其中一个对象修改这个String对象的时候，会新分配一块内存用来保存修改后的String对象并将其引用指向新的String对象，而原来的String对象因为还存在指向它的引用，所以不会被回收。这样，对于String而言，虽然是复制的引用，但是当修改值的时候，并不会改变被复制对象的值。**所以在使用克隆时，我们可以将 String类型视为基本类型，只需浅克隆即可。**


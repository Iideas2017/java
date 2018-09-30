# Java中的克隆方法

　　Java的所有类都是从java.lang.Object类继承而来的，而Object类提供protected Object clone\(\)方法对对象进行复制，子类当然也可以把这个方法置换掉，提供满足自己需要的复制方法。对象的复制有一个基本问题，就是对象通常都有对其他的对象的引用。当使用Object类的clone\(\)方法来复制一个对象时，此对象对其他对象的引用也同时会被复制一份

　　Java语言提供的Cloneable接口只起一个作用，就是在运行时期通知Java虚拟机可以安全地在这个类上使用clone\(\)方法。通过调用这个clone\(\)方法可以得到一个对象的复制。由于Object类本身并不实现Cloneable接口，因此如果所考虑的类没有实现Cloneable接口时，调用clone\(\)方法会抛出CloneNotSupportedException异常。

## 克隆满足的条件

　　clone\(\)方法将对象复制了一份并返还给调用者。所谓“复制”的含义与clone\(\)方法是怎么实现的。一般而言，clone\(\)方法满足以下的描述：

　　（1）对任何的对象x，都有：x.clone\(\)!=x。换言之，克隆对象与原对象不是同一个对象。

　　（2）对任何的对象x，都有：x.clone\(\).getClass\(\) == x.getClass\(\)，换言之，克隆对象与原对象的类型一样。

　　（3）如果对象x的equals\(\)方法定义其恰当的话，那么x.clone\(\).equals\(x\)应当成立的。

　　在JAVA语言的API中，凡是提供了clone\(\)方法的类，都满足上面的这些条件。JAVA语言的设计师在设计自己的clone\(\)方法时，也应当遵守着三个条件。一般来说，上面的三个条件中的前两个是必需的，而第三个是可选的。

## **浅克隆和深克隆** 

　　****无论你是自己实现克隆方法，还是采用Java提供的克隆方法，都存在一个浅度克隆和深度克隆的问题。

* 　　**浅度克隆**

　　只负责克隆按值传递的数据（比如基本数据类型、String类型），而不复制它所引用的对象，换言之，所有的对其他对象的引用都仍然指向原来的对象。

* 　　**深度克隆**

　　除了浅度克隆要克隆的值外，还负责克隆引用类型的数据。那些引用其他对象的变量将指向被复制过的新对象，而不再是原有的那些被引用的对象。换言之，深度克隆把要复制的对象所引用的对象都复制了一遍，而这种对被引用到的对象的复制叫做间接复制。  
****

　　深度克隆要深入到多少层，是一个不易确定的问题。在决定以深度克隆的方式复制一个对象的时候，必须决定对间接复制的对象时采取浅度克隆还是继续采用深度克隆。因此，在采取深度克隆时，需要决定多深才算深。此外，在深度克隆的过程中，很可能会出现循环引用的问题，必须小心处理。

## 利用序列化实现深度克隆

　　把对象写到流里的过程是序列化\(Serialization\)过程；而把对象从流中读出来的过程则叫反序列化\(Deserialization\)过程。应当指出的是，写到流里的是对象的一个拷贝，而原对象仍然存在于JVM里面。

　　在Java语言里深度克隆一个对象，常常可以先使对象实现Serializable接口，然后把对象（实际上只是对象的拷贝）写到一个流里（序列化），再从流里读回来（反序列化），便可以重建对象。

```text
    public  Object deepClone() throws IOException, ClassNotFoundException{
        //将对象写到流里
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bos);
        oos.writeObject(this);
        //从流里读回来
        ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bis);
        return ois.readObject();
    }
```

[![&#x590D;&#x5236;&#x4EE3;&#x7801;](http://common.cnblogs.com/images/copycode.gif)](javascript:void%280%29;)

　　这样做的前提就是对象以及对象内部所有引用到的对象都是可序列化的，否则，就需要仔细考察那些不可序列化的对象可否设成transient，从而将之排除在复制过程之外。

　　浅度克隆显然比深度克隆更容易实现，因为Java语言的所有类都会继承一个clone\(\)方法，而这个clone\(\)方法所做的正式浅度克隆。

　　有一些对象，比如线程\(Thread\)对象或Socket对象，是不能简单复制或共享的。不管是使用浅度克隆还是深度克隆，只要涉及这样的间接对象，就必须把间接对象设成transient而不予复制；或者由程序自行创建出相当的同种对象，权且当做复制件使用。

## 孙大圣的身外身法术

　　孙大圣的身外身本领如果在Java语言里使用原型模式来实现的话，会怎么样呢？首先，齐天大圣\(The Greatest Sage\)即TheGreatestSage类扮演客户角色。齐天大圣持有一个猢狲（Monkey）的实例，而猢狲就是大圣本尊。Monkey类具有继承自java.lang.Object的clone\(\)方法，因此，可以通过调用这个克隆方法来复制一个Monkey实例。

　　孙大圣本人用TheGreatestSage类代表[![&#x590D;&#x5236;&#x4EE3;&#x7801;](http://common.cnblogs.com/images/copycode.gif)](javascript:void%280%29;)

```text
public class TheGreatestSage {
    private Monkey monkey = new Monkey();
    
    public void change(){
        //克隆大圣本尊
        Monkey copyMonkey = (Monkey)monkey.clone();
        System.out.println("大圣本尊的生日是：" + monkey.getBirthDate());
        System.out.println("克隆的大圣的生日是：" + monkey.getBirthDate());
        System.out.println("大圣本尊跟克隆的大圣是否为同一个对象 " + (monkey == copyMonkey));
        System.out.println("大圣本尊持有的金箍棒 跟 克隆的大圣持有的金箍棒是否为同一个对象？ " + (monkey.getStaff() == copyMonkey.getStaff()));
    }
    
    public static void main(String[]args){
        TheGreatestSage sage = new TheGreatestSage();
        sage.change();
    }
}
```

[![&#x590D;&#x5236;&#x4EE3;&#x7801;](http://common.cnblogs.com/images/copycode.gif)](javascript:void%280%29;)

　　大圣本尊由Monkey类代表，这个类扮演具体原型角色：[![&#x590D;&#x5236;&#x4EE3;&#x7801;](http://common.cnblogs.com/images/copycode.gif)](javascript:void%280%29;)

```text
public class Monkey implements Cloneable {
    //身高
    private int height;
    //体重
    private int weight;
    //生日
    private Date birthDate;
    //金箍棒
    private GoldRingedStaff staff;
    /**
     * 构造函数
     */
    public Monkey(){
        this.birthDate = new Date();
        this.staff = new GoldRingedStaff();
    }
    /**
     * 克隆方法
     */
    public Object clone(){
        Monkey temp = null;
        try {
            temp = (Monkey) super.clone();
        } catch (CloneNotSupportedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } finally {
            return temp;
        }
    }
    public int getHeight() {
        return height;
    }
    public void setHeight(int height) {
        this.height = height;
    }
    public int getWeight() {
        return weight;
    }
    public void setWeight(int weight) {
        this.weight = weight;
    }
    public Date getBirthDate() {
        return birthDate;
    }
    public void setBirthDate(Date birthDate) {
        this.birthDate = birthDate;
    }
    public GoldRingedStaff getStaff() {
        return staff;
    }
    public void setStaff(GoldRingedStaff staff) {
        this.staff = staff;
    }
    
}
```

[![&#x590D;&#x5236;&#x4EE3;&#x7801;](http://common.cnblogs.com/images/copycode.gif)](javascript:void%280%29;)

　　大圣还持有一个金箍棒的实例，金箍棒类GoldRingedStaff:[![&#x590D;&#x5236;&#x4EE3;&#x7801;](http://common.cnblogs.com/images/copycode.gif)](javascript:void%280%29;)

```text
public class GoldRingedStaff {
    private float height = 100.0f;
    private float diameter = 10.0f;
    /**
     * 增长行为，每次调用长度和半径增加一倍
     */
    public void grow(){
        this.diameter *= 2;
        this.height *= 2;
    }
    /**
     * 缩小行为，每次调用长度和半径减少一半
     */
    public void shrink(){
        this.diameter /= 2;
        this.height /= 2;
    }
}
```

[![&#x590D;&#x5236;&#x4EE3;&#x7801;](http://common.cnblogs.com/images/copycode.gif)](javascript:void%280%29;)

　　当运行TheGreatestSage类时，首先创建大圣本尊对象，而后**浅度克隆**大圣本尊对象。程序在运行时打印出的信息如下：

![](data:image/png;base64,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)

　　可以看出，首先，复制的大圣本尊具有和原始的大圣本尊对象一样的birthDate，而本尊对象不相等，这表明他们二者是克隆关系；其次，复制的大圣本尊所持有的金箍棒和原始的大圣本尊所持有的金箍棒为同一个对象。这表明二者所持有的金箍棒根本是一根，而不是两根。

　　正如前面所述，继承自java.lang.Object类的clone\(\)方法是浅克隆。换言之，齐天大圣的所有化身所持有的金箍棒引用全都是指向一个对象的，这与《西游记》中的描写并不一致。要纠正这一点，就需要考虑使用**深克隆**。

　　为做到**深度克隆**，所有需要复制的对象都需要实现java.io.Serializable接口。

　　孙大圣的源代码：[![&#x590D;&#x5236;&#x4EE3;&#x7801;](http://common.cnblogs.com/images/copycode.gif)](javascript:void%280%29;)

```text
public class TheGreatestSage {
    private Monkey monkey = new Monkey();
    
    public void change() throws IOException, ClassNotFoundException{
        Monkey copyMonkey = (Monkey)monkey.deepClone();
        System.out.println("大圣本尊的生日是：" + monkey.getBirthDate());
        System.out.println("克隆的大圣的生日是：" + monkey.getBirthDate());
        System.out.println("大圣本尊跟克隆的大圣是否为同一个对象 " + (monkey == copyMonkey));
        System.out.println("大圣本尊持有的金箍棒 跟 克隆的大圣持有的金箍棒是否为同一个对象？ " + (monkey.getStaff() == copyMonkey.getStaff()));
    }
    
    public static void main(String[]args) throws IOException, ClassNotFoundException{
        TheGreatestSage sage = new TheGreatestSage();
        sage.change();
    }
}
```

[![&#x590D;&#x5236;&#x4EE3;&#x7801;](http://common.cnblogs.com/images/copycode.gif)](javascript:void%280%29;)

　　在大圣本尊Monkey类里面，有两个克隆方法，一个是clone\(\)，也即浅克隆；另一个是deepClone\(\)，也即深克隆。在深克隆方法里，大圣本尊对象（一个拷贝）被序列化，然后又被反序列化。反序列化的对象就成了一个深克隆的结果。

　　[![&#x590D;&#x5236;&#x4EE3;&#x7801;](http://common.cnblogs.com/images/copycode.gif)](javascript:void%280%29;)

```text
public class Monkey implements Cloneable,Serializable {
    //身高
    private int height;
    //体重
    private int weight;
    //生日
    private Date birthDate;
    //金箍棒
    private GoldRingedStaff staff;
    /**
     * 构造函数
     */
    public Monkey(){
        this.birthDate = new Date();
        staff = new GoldRingedStaff();
    }
    /**
     * 克隆方法
     */
    public Object clone(){
        Monkey temp = null;
        try {
            temp = (Monkey) super.clone();
        } catch (CloneNotSupportedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } finally {
            return temp;
        }
    }
    public  Object deepClone() throws IOException, ClassNotFoundException{
        //将对象写到流里
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bos);
        oos.writeObject(this);
        //从流里读回来
        ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bis);
        return ois.readObject();
    }
    public int getHeight() {
        return height;
    }
    public void setHeight(int height) {
        this.height = height;
    }
    public int getWeight() {
        return weight;
    }
    public void setWeight(int weight) {
        this.weight = weight;
    }
    public Date getBirthDate() {
        return birthDate;
    }
    public void setBirthDate(Date birthDate) {
        this.birthDate = birthDate;
    }
    public GoldRingedStaff getStaff() {
        return staff;
    }
    public void setStaff(GoldRingedStaff staff) {
        this.staff = staff;
    }
    
}
```

[![&#x590D;&#x5236;&#x4EE3;&#x7801;](http://common.cnblogs.com/images/copycode.gif)](javascript:void%280%29;)

　　可以看到，大圣本尊持有一个金箍棒（GoldRingedStaff）的实例。在大圣复制件里面，此金箍棒实例是原大圣本尊对象所持有的金箍棒对象的一个拷贝。在大圣本尊对象被序列化和反序列化时，它所持有的金箍棒对象也同时被序列化和反序列化，这使得复制的大圣的金箍棒和原大圣本尊对象所持有的金箍棒对象是两个独立的对象。[![&#x590D;&#x5236;&#x4EE3;&#x7801;](http://common.cnblogs.com/images/copycode.gif)](javascript:void%280%29;)

```text
public class GoldRingedStaff implements Serializable{
    private float height = 100.0f;
    private float diameter = 10.0f;
    /**
     * 增长行为，每次调用长度和半径增加一倍
     */
    public void grow(){
        this.diameter *= 2;
        this.height *= 2;
    }
    /**
     * 缩小行为，每次调用长度和半径减少一半
     */
    public void shrink(){
        this.diameter /= 2;
        this.height /= 2;
    }
}
```

[![&#x590D;&#x5236;&#x4EE3;&#x7801;](http://common.cnblogs.com/images/copycode.gif)](javascript:void%280%29;)

　　运行结果：

![](data:image/png;base64,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)

　　从运行的结果可以看出，大圣的金箍棒和他的身外之身的金箍棒是不同的对象。这是因为使用了深克隆，从而把大圣本尊所引用的对象也都复制了一遍，其中也包括金箍棒。


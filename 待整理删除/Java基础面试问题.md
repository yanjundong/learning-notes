

# 内部类问题

## 为什么需要使用内部类

- 内部类方法可以访问外部类的属性和方法；
- 内部类可以对同一个包中的其他类隐藏起来；
- 当想要定义一个回调函数且不想编写大量代码时，使用匿名内部类比较方便。

## 成员内部类

成员内部类是最普通的内部类，它的定义为位于另一个类的内部，举个例子：

```java
public class Circle {

    private double radius = 0;

    public Circle(double radius) {
        this.radius = radius;
    }

    public void method() {
        Draw draw = new Draw();
        draw.draw();
    }

    // 内部类
    class Draw {
        public void draw() {
            System.out.println("draw---" + radius);
        }
    }

    public static void main(String[] args) {
        Circle circle = new Circle(10.2);
        circle.method();
    }
}
```

类Draw像是Circle的一个成员，Circle称为外部类。成员内部类可以无条件访问外部类的所有成员属性和成员方法（包括private成员和静态成员）。

如果成员内部类拥有和外部类同名的成员变量或者方法时，会发生隐藏现象，即默认情况下访问的是成员内部类的成员。如果要访问外部类的同名成员，需要以下面的形式进行访问

![image-20201223165710315](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201223165710315.png)

```java
public class Circle {

    private double radius = 0;

    public Circle(double radius) {
        this.radius = radius;
    }

    // 内部类
    class Draw {

        double radius = 11.3;
        public void draw() {
            System.out.println("draw---" + Circle.this.radius);	// 访问外部类的属性
        }
    }

    public static void main(String[] args) {
        Circle circle = new Circle(10.2);
        Draw draw = circle.new Draw();	// 创建内部类对象
        draw.draw();
    }
}
```



成员内部类可以拥有private、protected、public以及包访问权限。可把它当做是外部类的一个成员变量来看。

## 局部内部类

局部内部类是定义在一个方法或者一个作用域里面的类，**它和成员内部类的区别在于局部内部类的作用域被限定在声明这个局部类的块中**。

![image-20201223170424238](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201223170424238.png)

![image-20201223171735137](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201223171735137.png)

### 为什么局部内部类只能访问它所属的方法中的final类型的变量呢？

局部变量的生命周期与局部内部类的对象的生命周期的不一致。**局部变量当所处的函数执行结束后就已经死亡了，不存在了，但是局部内部类对象还可能一直存在（只要有人还引用该对象），这是就会出现了一个悲剧的结果，局部内部类对象访问一个已不存在的局部变量（比如下图中的k）。**

那么，编译器是怎么实现的呢？编译器会将外部的final变量在编译阶段就作为内部类的成员变量写入内部类中，如下例子：

![image-20201223172428919](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201223172428919.png)

编译后：

![image-20201223172505238](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201223172505238.png)

## 匿名内部类

匿名内部类有以下特点： 

（1）没有名字（因为没有名字，没有构造函数）。 

（2）只能被实例化一次。 

（3）通常被声明在方法或代码块的内部，以一个带有分号的带有花括号结尾。 

（4）**匿名内部类也是不能有访问修饰符和static修饰符的。**

（5）匿名类的一个重要作用就是简化代码。

![image-20201223173126111](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201223173126111.png)

## 静态内部类

（1）静态内部类也是定义在另一个类里面的类，只不过在类的前面多了一个关键字static。 

（2）静态内部类是不需要依赖于外部类的，这点和类的静态成员属性有点类似。

（3）并且它不能使用外部类的非static成员变量或者方法。

![image-20201223173341501](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201223173341501.png)



## 静态内部类和成员内部类的不同

- 静态内部类不依赖于外部类对象；
- 不能从一个static内部类的一个对象访问一个外部类对象；



# 定义一个 Immutable Object

Java 语言目前还没有原生的不可变对象的支持，但在Java™ Tutorials中给出的如何定义一个不可变对象的方法。

1. 类中的属性不提供"setter"方法；

2. 类中所有的属性声明成private和final类型；

3. 类也声明成final的，以防止类被继承；

4. 如果有属性是引用类型的，也要防止引用类型的属性被调用方修改了，如通过构造器初始化所有成员，尤其是引用对象要进行深拷贝(deep copy，符合copy-on-write 原则)；

5. 如果确实需要实现 getter 方法，或者其他可能会返回内部状态的方法，也要深拷贝，创建私有的 copy。

![image-20201223174129204](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201223174129204.png)



![image-20201223174326994](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201223174326994.png)



# String类型的不可变

## 什么是String的不可变？

String不可变很简单，给一个已有字符串"abcd"第二次赋值成"abcedl"，不是在原内存地址上修改数据，而是重新指向一个新对象，新地址。

仅仅使用final对String类中使用的数组value进行修饰只能保证这个value不被改变，而不能防止value中的内容被改变。

![image-20201223175151591](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201223175151591.png)

所以String是不可变，关键是因为SUN公司的工程师，在后面所有String的方法里很小心的没有去动Array里的元素，没有暴露内部成员字段。private final char value[]这一句里，private的私有访问权限的作用都比final大。而且设计师还很小心地把整个String设成final禁止继承，避免被其他人继承后破坏。所以String是不可变的关键都在底层的实现，而不是一个final。考验的是工程师构造数据类型，封装数据的功力。

## String设置为不可变的好处

1. 保证安全性

   1. 传入某个函数中一个String类型的参数，如果String类型是可变的，这个函数中可能对String引用指向的内容做了变动，而这也许不是程序员的本意。

      ![image-20201223180118216](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201223180118216.png)

   2. String 不可变对HashSet和HashMap友好

      如果String类型是可变的，以String作为HashSet和HashMap中的key值时，在HashSet和HashMap中可能出现内存泄漏。而且可能破坏HashSet中键值的唯一性。

      ![image-20201223180215166](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201223180215166.png)

   3. String 不可变对线程安全友好

      在并发场景下，多个线程同时读一个资源，是不会引发竟态条件的。只有对资源做写操作才有危险。不可变对象不能被写，所以线程安全。

2. 常量池中复用String，可以节省内存空间

## 设计一个 immutable 类

设计一个不可变类应该遵循以下几点：

1、类的所有属性声明为private，去除掉所有的setter方法，防止外界直接对其进行修改

2、类的声明采用final进行修饰，保证没有父类对其修改

3、类的属性声明为final,如果对象类型为可变类型，应对其重新包装，重新new一个对象返回

![image-20201223180814483](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201223180814483.png)



## 为什么使用StringBuilder.append与直接使用String相加在性能上差异较大呢？

![image-20201223181041861](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201223181041861.png)



# Object类都有哪些函数？

```java
public native int hashCode()

public boolean equals(Object obj)

protected native Object clone() throws CloneNotSupportedException

public String toString()

public final native Class<?> getClass()

protected void finalize() throws Throwable {}

public final native void notify()

public final native void notifyAll()

public final native void wait(long timeout) throws InterruptedException

public final void wait(long timeout, int nanos) throws InterruptedException

public final void wait() throws InterruptedException
```

## clone方法

`clone()` 是 `Object` 的 `protected` 方法，一个类不显式去重写 `clone()`，其它类就不能直接去调用该类实例的 `clone()` 方法，并且还必须要实现 `Cloneable` 接口。

使用 `clone()` 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。《Effective Java》 书上讲到，最好不要去使用 `clone()`，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象。

```java
public class CloneConstructorExample {

    private int[] arr;

    public CloneConstructorExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public CloneConstructorExample(CloneConstructorExample original) {
        arr = new int[original.arr.length];
        for (int i = 0; i < original.arr.length; i++) {
            arr[i] = original.arr[i];
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }
}
CloneConstructorExample e1 = new CloneConstructorExample();
CloneConstructorExample e2 = new CloneConstructorExample(e1);
e1.set(2, 222);
System.out.println(e2.get(2)); // 2
```



# Java对象进行运程传输时需要进行序列化，如果实现？

**可以选用相应的序列化机制实现，比如java原生的对象序列化机制或者json、protocolBuffer等语言无关的序列化机制都可以的，**序列化机制要解决的关键问题就是如何序列化对象中引用的其它对象。

使用java中原生的对象序列化机制需要实现serializable或者externalizable接口，如果一个被序列化的对象其父类没有实现serializable接口或者externalizable接口，那么其父类相应的成员变量的属性都不会被序列化。

在序列化时类的哪些字段会参与到序列化中呢？其实有两种方式决定哪些字段会被序列化，

1. 默认方式，Java对象中的非静态和非transient的字段都会被定义为需要序列的字段。

2. 另外一种方式是通过 ObjectStreamField 数组来声明类需要序列化的对象。

   ```java
   public class A implements Serializable {
       String name;
       String password
   
       private static final ObjectStreamField[] serialPersistentFields
                    = {new ObjectStreamField("name", String.class)};
    }
   ```

**参考：**[细看Java序列化机制](https://juejin.im/post/5a7111535188257350518592#heading-1)




















































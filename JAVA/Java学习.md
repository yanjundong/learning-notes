# 一、面向对象基础

## 1、关键字

### 1.1 final

#### 数据

声明数据为常量，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量。

- 对于基本类型，final 使数值不变；
- 对于引用类型，final 使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。

```java
final int x = 1;
// x = 2;  // cannot assign value to final variable 'x'
final A y = new A();
y.a = 1;
```

#### 方法

声明方法不能被子类重写。

private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是重写基类方法，而是在子类中定义了一个新的方法。

#### 类

声明类不允许被继承。

### 1.2 static

#### 静态字段

在一个 `class` 中定义的字段，我们称之为实例字段。实例字段的特点是：每个实例都有自己独立的字段，各个实例的同名字段互不影响。

另外，还有一种特殊的，用`static`修饰的字段，称为静态字段：`static field`。

 静态字段只有一个共享“空间”，所有实例都会共享该字段。举个例子：

```java
/*Person.java*/
class Person {
    public String name;
    public int age;

    public static int number;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Person ming = new Person("Xiao Ming", 12);
        Person hong = new Person("Xiao Hong", 15);
        ming.number = 88;
        System.out.println(hong.number);	//88
        hong.number = 99;
        System.out.println(ming.number);	//99
    }
}
```

对于静态字段来说，该字段公共属于所有的实例。

在Java中，推荐通过`类名.静态字段`来访问静态对象，例如：`Person.number`。

#### 静态方法

用`static`修饰的方法称为静态方法。

> 静态方法在类加载的时候就存在了，它不依赖于任何实例。所以静态方法必须有实现，不能是抽象方法。
>
> ```java
> public abstract class A {
>     public static void func1(){
>     }
>     // public abstract static void func2();  // Illegal combination of modifiers: 'abstract' and 'static'
> }
> ```

调用实例方法必须通过一个实例变量，而调用静态方法则不需要实例变量，通过类名就可以直接调用。例如：

```java
/*Person.java*/
class Person {
    public static int number;

    public static void setNumber(int value) {
        number = value;
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Person.setNumber(99);
        System.out.println(Person.number);
    }
}
```

> 因为静态方法属于 `class` 而不是属于任何实例，因此，静态方法内部，不能访问 `this` 和 `super`变量，也不能访问实例字段和普通方法，只能访问静态字段和静态方法。

静态方法经常用于工具类。例如：

- `Arrays.sort()`
- `Math.random()`

#### 接口的静态字段

因为`interface`是一个纯抽象类，所以它不能定义实例字段。但是，`interface`是可以有静态字段的，并且静态字段必须为`final`类型：

```java
public interface Person {
    public static final int MALE = 1;
    public static final int FEMALE = 2;
}
```

#### 初始化顺序

静态变量和静态语句块优先于实例变量和普通语句块，静态变量和静态语句块的初始化顺序取决于它们在代码中的顺序。

```java
public static String staticField = "静态变量";
```

```java
static {
    System.out.println("静态语句块");
}
```

```java
public String field = "实例变量";
```

```java
{
    System.out.println("普通语句块");
}
```

最后才是构造函数的初始化。

```java
public InitialOrderTest() {
    System.out.println("构造函数");
}
```

存在继承的情况下，初始化顺序为：

- 父类（静态变量、静态语句块）
- 子类（静态变量、静态语句块）
- 父类（实例变量、普通语句块）
- 父类（构造函数）
- 子类（实例变量、普通语句块）
- 子类（构造函数）

## 2、作用域

### 2.1 public

定义为`public`的`class`、`interface`可以被其他任何类访问。

定义为`public`的`field`、`method`可以被其他类访问，前提是首先有访问`class`的权限。

### 2.2 private

定义为 `private` 的`method`、`field`不能被其他类访问，它的访问权限被限定在`class`的内部，而且与方法声明顺序*无关*。因此推荐把`private`方法放到后面。

### 2.3 protectd

`protected`作用于继承关系。定义为`protected`的字段和方法可以被子类访问，以及子类的子类。

```java
package abc;

public class Hello {
    // protected方法:
    protected void hi() {
    }
}
```

```java
package xyz;

class Main extends Hello {
    void foo() {
        Hello h = new Hello();
        // 因为继承了Hello，可以访问protected方法:
        h.hi();
    }
}
```

### 2.4 package

最后，包作用域是指一个类允许访问同一个`package`的没有`public`、`private`修饰的`class`，以及没有`public`、`protected`、`private`修饰的字段和方法。

> 注意，包名必须完全一致，包没有父子关系，`com.apache`和`com.apache.abc`是不同的包。

### 2.5 final

用`final`修饰`class`可以阻止被继承；

修饰`method`可以阻止被子类覆写；

修饰`field`可以阻止被重新赋值，

修饰`局部变量`可以阻止被重新赋值。

## 3、classpath

`classpath`是JVM用到的一个环境变量，它用来指示JVM如何搜索`class`。

因为Java是编译型语言，源码文件是`.java`，而编译后的`.class`文件才是真正可以被JVM执行的字节码。因此，JVM需要知道，如果要加载一个`abc.xyz.Hello`的类，应该去哪搜索对应的`Hello.class`文件。

现在我们假设`classpath`是`.;C:\work\project1\bin;C:\shared`，当JVM在加载`abc.xyz.Hello`这个类时，会依次查找：

- <当前目录>\abc\xyz\Hello.class
- C:\work\project1\bin\abc\xyz\Hello.class
- C:\shared\abc\xyz\Hello.class

> 第一个 `.`  代表当前目录。
>
> 如果JVM在某个路径下找到了对应的`class`文件，就不再往后继续搜索。如果所有路径下都没有找到，就报错。

```sh
vi ~/.bash_profile
export CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.
```

因为Java是编译型语言，源码文件是`.java`，而编译后的`.class`文件才是真正可以被JVM执行的字节码。因此，JVM需要知道，如果要加载一个`abc.xyz.Hello`的类，应该去哪搜索对应的`Hello.class`文件。

现在我们假设`classpath`是`.;C:\work\project1\bin;C:\shared`，当JVM在加载`abc.xyz.Hello`这个类时，会依次查找：

- <当前目录>\abc\xyz\Hello.class
- C:\work\project1\bin\abc\xyz\Hello.class
- C:\shared\abc\xyz\Hello.class

> 第一个 `.`  代表当前目录。
>
> 如果JVM在某个路径下找到了对应的`class`文件，就不再往后继续搜索。如果所有路径下都没有找到，就报错。

因此不推荐在系统环境变量中设置`classpath `，那样会污染整个系统环境，而应该在启动JVM时设置`classpath`变量。

在IDEA中，会自动传入参数。

# 二、Java核心类

## 1、基本数据类型

### 基本类型

- byte/8
- char/16
- short/16
- int/32
- float/32
- long/64
- double/64
- boolean/~

boolean 只有两个值：true、false，可以使用 1 bit 来存储，但是具体大小没有明确规定。JVM 会在编译时期将 boolean 类型的数据转换为 int，使用 1 来表示 true，0 表示 false。JVM 支持 boolean 数组，但是是通过读写 byte 数组来实现的。

### 包装类型

基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成。

```java
Integer x = 2;     // 装箱 调用了 Integer.valueOf(2)
int y = x;         // 拆箱 调用了 X.intValue()
```

### 缓存池

> `new Integer(123)` 与` Integer.valueOf(123) `的区别

- `new Integer(123)` 每次都会新建一个对象；
- `Integer.valueOf(123)` 会使用缓存池中的对象，多次调用会取得同一个对象的引用。

```java
Integer i1 = Integer.valueOf(123);
Integer i2 = Integer.valueOf(123);
System.out.println(i1 == i2);   //true

Integer i5 = new Integer(123);
Integer i6 = new Integer(123);
System.out.println(i5 == i6);   //false
```

`Integer.valueOf(123)`的实现比较简单，就是先判断 value 是否在`IntegerCache` 的范围中，如果在的话就从缓存池中获取，否则就new一个新的对象。

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

缓存池`IntegerCache` 的实现如下：

```java
private static class IntegerCache {
    static final int low = -128; //最小值为-128
    static final int high;	//最大值默认为127，但可以通过jvm的参数来指定
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        // 直接在cache中存放 low - high 的Integer对象
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```

正是因为`IntegerCache` 的最大范围，所以会出现如下情况：

```java
Integer i1 = Integer.valueOf(123);
Integer i2 = Integer.valueOf(123);
System.out.println(i1 == i2);   //true

Integer i3 = Integer.valueOf(1234);
Integer i4 = Integer.valueOf(1234);
System.out.println(i3 == i4);   //大于127，缓存池中没有，为false
```

基本类型对应的缓冲池如下：

- boolean values true and false
- all byte values
- short values between -128 and 127
- int values between -128 and 127
- char in the range \u0000 to \u007F

## 2、String

### 概览

String 被声明为 final，因此它不可被继承。

在 Java 8 中，String 内部使用 char 数组存储数据。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```

在 Java 9 之后，String 类的实现改用 byte 数组存储字符串，同时使用 `coder` 来标识使用了哪种编码。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final byte[] value;

    /** The identifier of the encoding used to encode the bytes in {@code value}. */
    private final byte coder;
}
```

value 数组被声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组。并且 String 内部没有改变 value 数组的方法，因此可以保证 String 不可变。

### String不可变的好处

1. 可以缓存 hash 值。因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

2. 如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。

   ![image-20201120233628604](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201120233628604.png)

3. String 不可变性天生具备线程安全，可以在多个线程中安全地使用。

## 3、Object

### 概览

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

### equals()

**1. 等价关系**

两个对象具有等价关系，需要满足以下五个条件：

Ⅰ 自反性

```
x.equals(x); // true
```

Ⅱ 对称性

```
x.equals(y) == y.equals(x); // true
```

Ⅲ 传递性

```
if (x.equals(y) && y.equals(z))
    x.equals(z); // true;
```

Ⅳ 一致性

多次调用 equals() 方法结果不变

```
x.equals(y) == x.equals(y); // true
```

Ⅴ 与 null 的比较

对任何不是 null 的对象 x 调用 x.equals(null) 结果都为 false

```
x.equals(null); // false;
```

**2. 等价与相等**

- 对于基本类型，== 判断两个值是否相等，基本类型没有 equals() 方法。
- 对于引用类型，== 判断两个变量是否引用同一个对象，而 equals() 判断引用的对象是否等价。

```
Integer x = new Integer(1);
Integer y = new Integer(1);
System.out.println(x.equals(y)); // true
System.out.println(x == y);      // false
```

**3. 实现**

- 检查是否为同一个对象的引用，如果是直接返回 true；
- 检查是否是同一个类型，如果不是，直接返回 false；
- 将 Object 对象进行转型；
- 判断每个关键域是否相等。

```
public class EqualExample {

    private int x;
    private int y;
    private int z;

    public EqualExample(int x, int y, int z) {
        this.x = x;
        this.y = y;
        this.z = z;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        EqualExample that = (EqualExample) o;

        if (x != that.x) return false;
        if (y != that.y) return false;
        return z == that.z;
    }
}
```

### hashCode()

- `hashCode()` 返回哈希值，而 `equals()` 是用来判断两个对象是否等价。

- 等价的两个对象散列值一定相同，但是散列值相同的两个对象不一定等价。

- 在覆盖 `equals()` 方法时应当总是覆盖 `hashCode()` 方法，保证等价的两个对象哈希值也相等。

- `HashSet` 和 `HashMap` 等集合类使用了 `hashCode()` 方法来计算对象应该存储的位置，因此要将对象添加到这些集合类中，需要让对应的类实现 `hashCode()` 方法。

  > 下面的代码中，新建了两个等价的对象，并将它们添加到 `HashSet` 中。我们希望将这两个对象当成一样的，只在集合中添加一个对象。但是 `EqualExample` 没有实现 `hashCode()` 方法，因此这两个对象的哈希值是不同的，最终导致集合添加了两个等价的对象。
  >
  > ```java
  > EqualExample e1 = new EqualExample(1, 1, 1);
  > EqualExample e2 = new EqualExample(1, 1, 1);
  > System.out.println(e1.equals(e2)); // true
  > HashSet<EqualExample> set = new HashSet<>();
  > set.add(e1);
  > set.add(e2);
  > System.out.println(set.size());   // 2
  > ```

### toString()



### clone()

**1. cloneable**

`clone()` 是 `Object` 的 `protected` 方法，它不是` public`，一个类不显式去重写 `clone()`，其它类就不能直接去调用该类实例的 `clone()` 方法。

```java
public class CloneExample {
    private int a;
    private int b;
}
CloneExample e1 = new CloneExample();
// CloneExample e2 = e1.clone(); // 'clone()' has protected access in 'java.lang.Object'
```

重写 `clone()` 得到以下实现：

```java
public class CloneExample {
    private int a;
    private int b;

    @Override
    public CloneExample clone() throws CloneNotSupportedException {
        return (CloneExample)super.clone();
    }
}
CloneExample e1 = new CloneExample();
try {
    CloneExample e2 = e1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
// java.lang.CloneNotSupportedException: CloneExample
```

以上抛出了 `CloneNotSupportedException`，这是因为 `CloneExample` 没有实现 `Cloneable` 接口。

应该注意的是，`clone()` 方法并不是 `Cloneable` 接口的方法，而是 `Object` 的一个 `protected` 方法。`Cloneable` 接口只是规定，如果一个类没有实现 `Cloneable` 接口又调用了 `clone()` 方法，就会抛出 `CloneNotSupportedException`。

```java
public class CloneExample implements Cloneable {
    private int a;
    private int b;

    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

**2. clone() 的替代方案**

使用 `clone()` 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。Effective Java 书上讲到，最好不要去使用 `clone()`，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象。

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

# 三、异常处理

## 1、Java的异常

在计算机程序运行的过程中，总是会出现各种各样的错误。

有些错误是用户造成的，比如，希望用户输入一个`int`类型的年龄，但是用户的输入是`abc`：

```java
// 假设用户输入了abc：
String s = "abc";
int n = Integer.parseInt(s); // NumberFormatException!
```

还有一些错误是随机出现，并且永远不可能避免的。比如：

- 网络突然断了，连接不到远程服务器；
- 内存耗尽，程序崩溃了；

所以，一个健壮的程序必须处理各种各样的错误。

Java对于错误的处理是内置了一套异常处理机制，总是使用异常来表示错误。

异常是一种`class`，因此它本身就带有类型信息。异常可以在任何地方抛出，但是需要在上层捕获，这样就和方法调用分离了：

```java
try {
    String s = processFile(“C:\\test.txt”);
    // ok:
} catch (FileNotFoundException e) {
    // file not found:
} catch (SecurityException e) {
    // no read permission:
} catch (IOException e) {
    // io error:
} catch (Exception e) {
    // other error:
}
```



Java的异常是`class`，它的继承关系如下：

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ20200308-090814.png)

从继承关系可知：`Throwable`是异常体系的根，它继承自`Object`。`Throwable`有两个体系：`Error`和`Exception`，

`Error`表示严重的错误，程序对此一般无能为力，例如：

- `OutOfMemoryError`：内存耗尽
- `NoClassDefFoundError`：无法加载某个Class
- `StackOverflowError`：栈溢出

而`Exception`则是运行时的错误，它可以被捕获并处理。

某些异常是应用程序逻辑处理的一部分，应该捕获并处理。例如：

- `NumberFormatException`：数值类型的格式错误
- `FileNotFoundException`：未找到文件
- `SocketException`：读取网络失败

还有一些异常是程序逻辑编写不对造成的，应该修复程序本身。例如：

- `NullPointerException`：对某个`null`的对象调用方法或字段
- `IndexOutOfBoundsException`：数组索引越界

> 总结：
>
> - `Error`是十分严重的错误，程序对此无能为力，不需要捕获。
> - `RuntimeException`是运行时的错误，应该修改代码本身，不强制捕获。
> - 非`RuntimeException`（Checked Exception）是逻辑处理的错误，必须要捕获处理。

> 不推荐捕获了异常但不进行任何处理。

## 2、捕获异常

在Java中，凡是可能抛出异常的语句，都可以用`try ... catch`捕获。

### 2.1 多catch语句

可以使用多个`catch`语句，每个`catch`分别捕获对应的`Exception`及其子类。

JVM在捕获到异常后，会从上到下匹配`catch`语句，匹配到某个`catch`后，执行`catch`代码块，然后*不再*继续匹配。

```java
public static void main(String[] args) {
    try {
        process1();
        process2();
        process3();
    } catch (IOException e) {
        System.out.println(e);
    } catch (NumberFormatException e) {
        System.out.println(e);
    }
}
```

> 注意：因为JVM在配置到某个 `catch` 后，就不再继续匹配。因此，存在多个 `catch` 时，`catch` 的顺序很重要：子类必须写在前面。否则会永远捕获不到。例如：
>
> ```java
> public static void main(String[] args) {
>     try {
>         process1();
>         process2();
>         process3();
>     } catch (IOException e) {
>         System.out.println("IO error");
>     } catch (UnsupportedEncodingException e) { 
>       // UnsupportedEncodingException是IOException的子类，所以永远捕获不到
>         System.out.println("Bad encoding");
>     }
> }
> ```
>
> 

### 2.2 finally语句

我们我们希望无论是否有异常发生，都执行一些语句，例如关闭数据库等，可以使用 `finally` 语句。

`finally`语句块保证有无错误都会执行。

```java
public static void main(String[] args) {
    try {
        process1();
        process2();
        process3();
    } catch (UnsupportedEncodingException e) {
        System.out.println("Bad encoding");
    } catch (IOException e) {
        System.out.println("IO error");
    } finally {
        System.out.println("END");
    }
}
```

`finally`有几个特点：

1. `finally`语句不是必须的，可写可不写；
2. `finally`总是最后执行。

> 某些情况下，可以没有`catch`，只使用`try ... finally`结构

### 2.3 捕获多种异常

如果某些异常的处理代码相同，但是异常本身不存在继承关系，可以把它两用`|`合并到一起：

```java
public static void main(String[] args) {
    try {
        process1();
        process2();
        process3();
    } catch (IOException | NumberFormatException e) { // IOException或NumberFormatException
        System.out.println("Bad input");
    } catch (Exception e) {
        System.out.println("Unknown error");
    }
}
```

## 3、抛出异常

### 3.1 异常的传播

当某个方法抛出了异常时，如果当前方法没有捕获异常，异常就会被抛到上层调用方法，直到遇到某个`try ... catch`被捕获为止。

```java
public class Main {
    public static void main(String[] args) {
        try {
            process1();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    static void process1() {
        process2();
    }

    static void process2() {
        Integer.parseInt(null); // 会抛出NumberFormatException
    }
}
```

通过`printStackTrace()`可以打印出方法的调用栈，类似：

```java
java.lang.NumberFormatException: null
    at java.base/java.lang.Integer.parseInt(Integer.java:614)
    at java.base/java.lang.Integer.parseInt(Integer.java:770)
    at Main.process2(Main.java:16)
    at Main.process1(Main.java:12)
    at Main.main(Main.java:5)
```

上述信息表示：

- `NumberFormatException`是在`java.lang.Integer.parseInt`方法中被抛出的；
- 调用层次从上到下依次是：
  1. `main()`调用`process1()`；
  2. `process1()`调用`process2()`；
  3. `process2()`调用`Integer.parseInt(String)`；
  4. `Integer.parseInt(String)`调用`Integer.parseInt(String, int)`。

查看`Integer.java`源码可知，抛出异常的方法代码如下：

```java
public static int parseInt(String s, int radix) throws NumberFormatException {
    if (s == null) {
        throw new NumberFormatException("null");
    }
    ...
}
```

### 3.2 抛出异常

当发生错误时，例如，用户输入了非法的字符，我们就可以抛出异常。

如何抛出异常？参考`Integer.parseInt()`抛出异常的方法，抛出异常分两步：

1. 创建某个`Exception`的实例；
2. 用`throw`语句抛出。

例子：

```java
void process2(String s) {
    if (s==null) {
        throw new NullPointerException();
    }
}
```



如果一个方法捕获了某个异常后，又在`catch`子句中抛出新的异常，就相当于把抛出的异常类型“转换”了：

```java
public class Main {
    public static void main(String[] args) {
        try {
            process1();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    static void process1() {
        try {
            process2();
        } catch (NullPointerException e) {
            throw new IllegalArgumentException();
        }
    }

    static void process2() {
        throw new NullPointerException();
    }
}

```

在上面的代码中，当`process2()`抛出`NullPointerException`后，被`process1()`捕获，然后抛出`IllegalArgumentException()`。最后，在`main()`中捕获`IllegalArgumentException`并打印异常栈，如下：

```java
java.lang.IllegalArgumentException
	at Main.process1(Main.java:19)
	at Main.main(Main.java:9)
```

这说明新的异常丢失了原始异常信息，我们已经看不到原始异常`NullPointerException`的信息了。

为了能追踪到完整的异常栈，在构造异常的时候，把原始的`Exception`实例传进去，新的`Exception`就可以持有原始`Exception`信息。如下代码：

```java
public class Main {
    //...

    static void process1() {
        try {
            process2();
        } catch (NullPointerException e) {
            throw new IllegalArgumentException(e);
        }
    }

    //...
}
```

这次，打印出来的异常栈类似：

```java
java.lang.IllegalArgumentException: java.lang.NullPointerException
	at Main.process1(Main.java:19)
	at Main.main(Main.java:9)
Caused by: java.lang.NullPointerException
	at Main.process2(Main.java:24)
	at Main.process1(Main.java:17)
	... 1 more
```

由输出可以知道：

- `Caused by: Xxx`知道造成问题的根源是 `NullPointerException`，是在`Main.process2()`方法抛出的。

## 4、自定义异常

Java标准库定义的常用异常包括：

```ascii
Exception
│
├─ RuntimeException
│  │
│  ├─ NullPointerException
│  │
│  ├─ IndexOutOfBoundsException
│  │
│  ├─ SecurityException
│  │
│  └─ IllegalArgumentException
│     │
│     └─ NumberFormatException
│
├─ IOException
│  │
│  ├─ UnsupportedCharsetException
│  │
│  ├─ FileNotFoundException
│  │
│  └─ SocketException
│
├─ ParseException
│
├─ GeneralSecurityException
│
├─ SQLException
│
└─ TimeoutException
```

在需要排除异常时，尽量使用已定义的异常类型。

在一个大型项目中，可以**自定义新的异常类型**，但是，保持一个合理的异常继承体系是非常重要的。

一个常见的做法是自定义一个`BaseException`作为“根异常”，然后，派生出各种业务类型的异常。

`BaseException`需要从一个适合的`Exception`派生，通常建议从`RuntimeException`派生：

```java
public class BaseException extends RuntimeException {
}
```

其他业务类型的异常就可以从`BaseException`派生：

```java
public class UserNotFoundException extends BaseException {
}

public class LoginFailedException extends BaseException {
}

//...
```

自定义的`BaseException`应该提供多个构造方法：

```java
public class BaseException extends RuntimeException {
    public BaseException() {
        super();
    }

    public BaseException(String message, Throwable cause) {
        super(message, cause);
    }

    public BaseException(String message) {
        super(message);
    }

    public BaseException(Throwable cause) {
        super(cause);
    }
}
```

上述构造方法实际上都是原样照抄`RuntimeException`。这样，抛出异常的时候，就可以选择合适的构造方法。通过IDE可以根据父类快速生成子类的构造方法。

## 5、日志输出

[SLF4J和Logback的使用](https://blog.csdn.net/weixin_40971059/article/details/104673016)

# 四、反射

## 1、定义

java反射机制是说：在运行状态中，对于任意的一个类，我们都能知道它的所有属性和方法；对于任意的一个对象，我们都能够调用它的所有方法和得到所有属性。

这种动态调用方法和动态获取属性的功能成为java语言的反射机制。

实际上，我们创建的每一个类都是Class对象，称之为类对象。

## 2、Class类

除了`int`等基本类型外，Java的其他类型全部都是`class`（包括`interface`）。例如：

- `String`
- `Object`
- `Runnable`
- `Exception`
- ...

`class` 是由JVM在执行过程中动态加载的。JVM在第一次读取到一种`class`类型时，将其加载进内存。

每加载一种`class`，JVM就为其创建一个`Class`类型的实例，并关联起来。注意：这里的`Class`类型是一个名叫`Class`的`class`。它长这样：

```java
public final class Class {
    private Class() {}
}
```

> 这个`Class`实例是JVM内部创建的，可以发现`Class`类的构造方法是`private`，只有JVM能创建`Class`实例，我们自己的Java程序是无法创建`Class`实例的。

以`String`类为例，当JVM加载`String`类时，它首先读取`String.class`文件到内存，然后，为`String`类创建一个`Class`实例并关联起来：

```java
Class cls = new Class(String);
```

所以，JVM持有的每个`Class`实例都指向一个数据类型（`class`或`interface`）：

```ascii
┌───────────────────────────┐
│      Class Instance       │──────> String
├───────────────────────────┤
│name = "java.lang.String"  │
└───────────────────────────┘
┌───────────────────────────┐
│      Class Instance       │──────> Random
├───────────────────────────┤
│name = "java.util.Random"  │
└───────────────────────────┘
┌───────────────────────────┐
│      Class Instance       │──────> Runnable
├───────────────────────────┤
│name = "java.lang.Runnable"│
└───────────────────────────┘
```

一个`Class`实例包含了该`class`的所有完整信息，包括类名、包名、父类、实现的接口、所有方法、字段等：

```ascii
┌───────────────────────────┐
│      Class Instance       │──────> String
├───────────────────────────┤
│name = "java.lang.String"  │
├───────────────────────────┤
│package = "java.lang"      │
├───────────────────────────┤
│super = "java.lang.Object" │
├───────────────────────────┤
│interface = CharSequence...│
├───────────────────────────┤
│field = value[],hash,...   │
├───────────────────────────┤
│method = indexOf()...      │
└───────────────────────────┘
```

由上解释，可以知道 如果获取了某个`Class`实例，我们就可以通过这个`Class`实例获取到该实例对应的`class`的所有信息。

这种通过`Class`实例获取`class`信息的方法就称为反射（Reflection）。

### 2.1 获取Class实例

```java
/*Student.java*/
package reflex;

public class Student extends Person {
    private int id;
    private String name;
    private String gender;
    private String address;
    private Date date;

    public Student(){

    }

    public Student(String name){
        this.name = name;
    }

    private Student(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public Student(String name, String address){
        this.address = address;
    }

    private Student(String name, String gender, String address, Date date) {
        this.name = name;
        this.gender = gender;
        this.address = address;
        this.date = date;
    }
		
  	//getter和setter方法
  	//toString方法
}
```



```java
/*ReflexTest.java*/
package reflex;

import org.junit.Test;

public class ReflexTest {
    //获取类对象
    @Test
    public void m1() throws ClassNotFoundException {
        Student student = new Student();
        //方法一
        System.out.println(Student.class);
        //方法二
        System.out.println(student.getClass());
        //方法三
        String className = "reflex.Student";
        System.out.println(Class.forName(className));
    }
  	//...
}
```

```java
//输出结果
class reflex.Student
class reflex.Student
class reflex.Student
```

> 因为`Class`实例在JVM中是唯一的，所以，上述方法获取的`Class`实例是同一个实例。
>
> `Class`实例比较和`instanceof`的差别：
>
> ```java
> Integer n = new Integer(123);
> 
> boolean b1 = n instanceof Integer; // true，因为n是Integer类型
> boolean b2 = n instanceof Number; // true，因为n是Number类型的子类
> 
> boolean b3 = n.getClass() == Integer.class; // true，因为n.getClass()返回Integer.class
> boolean b4 = n.getClass() == Number.class; // false，因为Integer.class!=Number.class
> ```
>
> 

如果获取到了一个`Class`实例，我们就可以通过该`Class`实例来创建对应类型的实例：

```java
// 获取String的Class实例:
Class cls = String.class;
// 创建一个String实例:
String s = (String) cls.newInstance();
```

上述代码相当于`new String()`。通过`Class.newInstance()`可以创建类实例，它的局限是：只能调用`public`的无参数构造方法。带参数的构造方法，或者非`public`的构造方法都无法通过`Class.newInstance()`被调用。

### 2.2 JVM的动态加载

JVM在执行Java程序的时候，并不是一次性把所有用到的class全部加载到内存，而是第一次需要用到class时才加载。例如：

```java
// Main.java
public class Main {
    public static void main(String[] args) {
        if (args.length > 0) {
            create(args[0]);
        }
    }

    static void create(String name) {
        Person p = new Person(name);
    }
}
```

当执行`Main.java`时，由于用到了`Main`，因此，JVM首先会把`Main.class`加载到内存。然而，并不会加载`Person.class`，除非程序执行到`create()`方法，JVM发现需要加载`Person`类时，才会首次加载`Person.class`。如果没有执行`create()`方法，那么`Person.class`根本就不会被加载。

这就是JVM动态加载`class`的特性。

## 3、获取类成员

### 3.1 获取类的构造方法

```java
//获取类的所有public类型的构造方法
    @Test
    public void m2(){
        Student student = new Student();
        Class c = student.getClass();
        //获取类的所有构造方法
        Constructor[] constructors = c.getDeclaredConstructors();
        /**
         * 通过getDeclaredConstructors方法可以返回类的构造方法，返回的是一个数组
         * 通过getModifiers可以得到构造方法的类型，返回的是数字，1-public 2-private
         * 通过getParameterTypes可以得到某构造方法的所有参数类型
         */
        for(int i = 0;i < constructors.length;++i){
          	//获取构造方法的类型
            System.out.print(Modifier.toString(constructors[i].getModifiers()) + " " + "参数:");  
          	//获取构造方法的参数类型
            Class[] parameterTypes = constructors[i].getParameterTypes();  
            for(int j = 0 ;j < parameterTypes.length;++j){
                System.out.print(parameterTypes[j].getName()+ " ");
            }
            System.out.println("");
        }
    }
```

```java
//运行结果
private 参数:java.lang.String java.lang.String 
public 参数:java.lang.String 
public 参数:
```

#### 获取类的特定的构造方法

```java
//返回特定的构造方法
    @Test
    public void m3() throws NoSuchMethodException {
        Student stu = new Student();
        Class[] c = {String.class,String.class};
        Constructor constructor = stu.getClass().getConstructor(c);
        System.out.print(Modifier.toString(constructor.getModifiers()) + " " + "参数:");
        Class[] parameterTypes = constructor.getParameterTypes();
        for(int j = 0 ;j < parameterTypes.length;++j){
            System.out.print(parameterTypes[j].getName()+ " ");
        }
        System.out.println("");
    }
```

```java
//运行结果
public 参数:java.lang.String java.lang.String 
```

#### 调用构造方法

```java
//调用构造方法
    @Test
    public void m4() throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        Student stu = new Student();

        //调用公有构造函数
        Class[] c1 = {String.class};
        Constructor constructor1 = stu.getClass().getDeclaredConstructor(c1);
        Student stu1 = (Student)constructor1.newInstance("yanjundong");
        System.out.println(stu1);

        //调用私有构造函数
        Class[] c2 = {int.class,String.class};
        Constructor constructor2 = stu.getClass().getDeclaredConstructor(c2);
        constructor2.setAccessible(true);  //多出的一步
        Student stu2 = (Student)constructor2.newInstance(1001,"yanjundong");
        System.out.println(stu2);
    }
```

```java
//运行结果
Student{id=0, name='yanjundong', gender='null', address='null', date=null}
Student{id=1001, name='yanjundong', gender='null', address='null', date=null}
```

> 总结：
>
> 通过Class实例获取Constructor的方法如下：
>
> - `getConstructor(Class...)`：获取某个`public`的`Constructor`；
> - `getDeclaredConstructor(Class...)`：获取某个`Constructor`（包括 `private` 的）；
> - `getConstructors()`：获取所有`public`的`Constructor`；
> - `getDeclaredConstructors()`：获取所有`Constructor`（包括 `private` 的）。

### 3.2 调用方法

可以通过`Class`实例获取所有`Method`信息。`Class`类提供了以下几个方法来获取`Method`：

- `Method getMethod(name, Class...)`：获取某个`public`的`Method`（包括父类）
- `Method getDeclaredMethod(name, Class...)`：获取当前类的某个`Method`（不包括父类）
- `Method[] getMethods()`：获取所有`public`的`Method`（包括父类）
- `Method[] getDeclaredMethods()`：获取当前类的所有`Method`（不包括父类）

```java
		@Test
    public void m5() throws NoSuchMethodException {
        Class stdClass = Student.class;
        Method setName = stdClass.getMethod("setName", String.class);
        System.out.println(setName);
        System.out.println("=================");
        Method[] methods = stdClass.getDeclaredMethods();
        for (Method method : methods) {
            System.out.println(method);
        }

    }
```

> 一个`Method`对象包含一个方法的所有信息：
>
> - `getName()`：返回方法名称，例如：`"getScore"`；
> - `getReturnType()`：返回方法返回值类型，也是一个Class实例，例如：`String.class`；
> - `getParameterTypes()`：返回方法的参数类型，是一个Class数组，例如：`{String.class, int.class}`；
> - `getModifiers()`：返回方法的修饰符，它是一个`int`，不同的bit表示不同的含义。

#### invoke调用方法

对`Method`实例调用`invoke`就相当于调用该方法，`invoke`的第一个参数是对象实例，即在哪个实例上调用该方法，后面的可变参数要与方法参数一致，否则将报错。

```java
		@Test
    public void m6() throws Exception {
       // String对象:
        String s = "Hello world";
        // 获取String substring(int)方法，参数为int:
        Method m = String.class.getMethod("substring", int.class);
        // 在s对象上调用该方法并获取结果:
        String r = (String) m.invoke(s, 6);
        // 打印调用结果:
        System.out.println(r);
    }
```

#### 调用静态方法

如果获取到的Method表示一个静态方法，调用静态方法时，由于无需指定实例对象，所以`invoke`方法传入的第一个参数永远为`null`。我们以`Integer.parseInt(String)`为例：

```java
		@Test
    public void m7() throws Exception{
       /// 获取Integer.parseInt(String)方法，参数为String:
        Method m = Integer.class.getMethod("parseInt", String.class);
        // 调用该静态方法并获取结果:
        Integer n = (Integer) m.invoke(null, "12345");
        // 打印调用结果:
        System.out.println(n);
    }
```

#### 调用类的私有方法

```java
//调用类的私有方法
    @Test
    public void m8() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        Student stu = new Student();

        Class[] p = {String.class};
        //通过getDeclaredMethod方法获取这个私有方法，第一个参数为方法名，第二个为参数类型
        Method method = stu.getClass().getDeclaredMethod("welcome", p);
        method.setAccessible(true);
        //通过invoke方法执行，第一个参数为类的实例，第二个参数为方法参数
        Object args[] = {"java反射测试"};
        method.invoke(stu,args);
    }
```

```java
//运行结果
java反射测试
```

### 3.3 访问字段

`Class`类提供了以下几个方法来获取字段：

- `Field getField(name)`：根据字段名获取某个`public`的`field`（包括父类）
- `Field getDeclaredField(name)`：根据字段名获取当前类的某个`field`（不包括父类）
- `Field[] getFields()`：获取所有`public`的`field`（包括父类）
- `Field[] getDeclaredFields()`：获取当前类的所有`field`（不包括父类）

> 一个`Field`对象包含了一个字段的所有信息：
>
> - `getName()`：返回字段名称，例如，`"name"`；
> - `getType()`：返回字段类型，也是一个`Class`实例，例如，`String.class`；
> - `getModifiers()`：返回字段的修饰符，它是一个`int`，不同的bit表示不同的含义。

#### 调用类的私有字段，并修改值

```java
//调用类的私有字段，并修改值
    @Test
    public void m9() throws NoSuchFieldException, IllegalAccessException {
        Student stu = new Student("zhangsan", "陕西省西安市");
        Field field = Student.class.getDeclaredField("name");
      	//不管这个字段是不是public，一律允许访问
        field.setAccessible(true);
      	//设置字段值，第一个Object参数是指定的实例，第二个Object参数是待修改的值
        field.set(stu, "张三");
        String name = field.get(stu).toString();
        System.out.println(name);
    }
```

```java
//运行结果
张三
```

### 3.4 获取继承关系

#### 获取父级的Class

```java
		@Test
    public void m10() throws NoSuchFieldException, IllegalAccessException {
        Class i = Integer.class;
        Class n = i.getSuperclass();
        System.out.println(n);
        Class o = n.getSuperclass();
        System.out.println(o);
        System.out.println(o.getSuperclass());
    }
```

```java
//运行结果
class java.lang.Number
class java.lang.Object
null
```

> Integer`的父类类型是`Number`，`Number`的父类是`Object`，`Object`的父类是`null。

#### 获取interface

由于一个类可能实现一个或多个接口，通过`Class`我们就可以查询到实现的接口类型。例如，查询`String`实现的接口：

```java
    @Test
    public void m11() {
        Class s = String.class;
        Class[] is = s.getInterfaces();
        for (Class i : is) {
            System.out.println(i);
        }
    }
```

```java
//运行结果
//Stirng实现了下面3个接口
interface java.io.Serializable
interface java.lang.Comparable
interface java.lang.CharSequence
```

> 要特别注意：`getInterfaces()`只返回当前类直接实现的接口类型，并不包括其父类实现的接口类型：

## 4、Fastjson

--还是爸爸的产品好用，简单、方便，虽然网上有人说没有Jackson稳定，

```java
//添加maven依赖
<!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>fastjson</artifactId>
      <version>1.2.62</version>
    </dependency>
```

### 4.1 将Java对象转化为JSON字符串

```java
		@Test
    public void m12() throws IOException {
        Student student = new Student();
        student.setId(1000);
        student.setAddress("XXXXXXXX");
        student.setName("zhangsan");
        student.setGender("M");
        student.setDate(new Date());
        
        String jsonString = JSON.toJSONString(student);
        System.out.println(jsonString);
    }
```

```java
//输出结果
{"address":"XXXXXXXX","date":1571925626901,"gender":"M","id":1000,"name":"zhangsan"}
```

### 4.2 将JSON字符串转化为Java对象

```java
		@Test
    public void m13() throws IOException {
        Student student = new Student();
        student.setId(1000);
        student.setAddress("XXXXXXXX");
        student.setName("zhangsan");
        student.setGender("M");
        student.setDate(new Date());
        //序列化
        String jsonString = JSON.toJSONString(student);
        System.out.println(jsonString);
        //反序列化
        Student student1 = JSON.parseObject(jsonString,Student.class);
        System.out.println(student1);
    }
```

```java
//输出结果
{"address":"XXXXXXXX","date":1583646100676,"gender":"M","id":1000,"name":"zhangsan"}
Student{id=1000, name='zhangsan', gender='M', address='XXXXXXXX', date=Sun Mar 08 13:41:40 CST 2020}
```

### 4.3 注解符的使用

在 `Student.java` 的字段上添加上注解，如下所示：

```java
/*Student.java*/
		@JSONField(serialize=false)
    private int id;
    @JSONField(ordinal = 1)
    private String name;
    @JSONField(ordinal = 2)
    private String gender;
    @JSONField(ordinal = 4)
    private String address;
    @JSONField(format="yyyy-MM-dd",name = "birthday",ordinal = 3)
    private Date date;

//可以使用serialize指定字段不序列化，使用deserialize指定字段不反序列化
//format参数用于格式化date属性，只作用于序列化
//使用ordinal可以指定字段的顺序
```

添加注解之后，`3.2 将JSON字符串转化为Java对象` 的测试输入如下：

```java
//输出结果
{"name":"zhangsan","gender":"M","birthday":"2020-03-08","address":"XXXXXXXX"}
Student{id=0, name='zhangsan', gender='M', address='XXXXXXXX', date=Sun Mar 08 00:00:00 CST 2020}
```



### 4.4 实现对value的过滤 

使用 `ContextValueFilter` 配置 JSON 转换，实现对value的过滤，

```java
    @Test
    public void m14(){
        Student student = new Student();
        student.setId(1000);
        student.setAddress("XXXXXXXX");
        student.setName("zhangsan");
        student.setGender("M");
        student.setDate(new Date());
        
        ContextValueFilter valueFilter = new ContextValueFilter() {
            @Override
            public Object process(BeanContext beanContext, Object o, String name, Object value) {
                if (name.equals("gender")) {
                    return "NOT TO DISCLOSE";
                }
                if (value.equals("zhangsan")) {
                    return ((String) value).toUpperCase();
                }else{
                    return value;
                }
            }
        };
        
        String jsonOutput = JSON.toJSONString(student, valueFilter);
        System.out.println(jsonOutput);
    }
```

```java
//输出结果
{"name":"ZHANGSAN","gender":"NOT TO DISCLOSE","birthday":"2019-10-25","address":"XXXXXXXX"}
```

### 4.5 实现对key的过滤

`NameFilter` 序列化时修改key；

`SerializeConfig` 内部是个map容器主要功能是配置并记录每种Java类型对应的序列化类;

```java
    @Test
    public void m5(){
        List<Student> studentList = new ArrayList<Student>();
      
        Student student = new Student();
        student.setId(1000);
        student.setAddress("XXXXXXXX");
        student.setName("zhangsan");
        student.setGender("M");
        student.setDate(new Date());
        studentList.add(student);
      
        Student student1 = new Student();
        student1.setId(1001);
        student1.setAddress("XXXX-XXXX");
        student1.setName("lisi");
        student1.setGender("W");
        student1.setDate(new Date());
        studentList.add(student1);

        //使用NameFilter过滤器处理字段名称
        NameFilter formatName = new NameFilter() {
            @Override
            public String process(Object o, String name, Object value) {
                if("address".equals(name))
                    return "address of home";
                else
                    return name;
            }
        };
      
      
        // 新创建的过滤器与Student类相关联，然后添加到全局实例，它是SerializeConfig类中的静态属性。
        SerializeConfig.getGlobalInstance().addFilter(Student.class,  nameFilter);
        //使用toJSONStringWithDateFormat而不是toJSONString，它可以更快速的格式化日期。
        //String jsonString = JSON.toJSONStringWithDateFormat(studentList, "yyyy-MM-dd");
        String jsonString = JSON.toJSONString(studentList);
        System.out.println(jsonString);



        /*可以实现同样的效果
        String jsonString1 = JSON.toJSONString(studentList, nameFilter);
        System.out.println(jsonString1);*/
    }
```

```java
//输出结果
[{"name":"zhangsan","gender":"M","birthday":"2020-03-08","address of home":"XXXXXXXX"},{"name":"lisi","gender":"W","birthday":"2020-03-08","address of home":"XXXX-XXXX"}]
```



# 五、泛型

## 1. 为什么要使用泛型程序设计

`泛型程序设计` 意味着编写的代码能够被很多不同类型的对象所重用。我们不需要为 `String` 和 `Integer` 对象分别设计不同的类。使用泛型，就可以像 `ArrayList` 一样可以聚集任何类型的对象。

```
List<String> strs = new ArrayList<>();
List<Integer> vlues = new ArrayList<>();
```

> 在增加泛型类之前，程序的泛型程序设计是通过继承来实现的。在 `ArrayList` 类中维护了一个 `Object` 数组，这样做的缺点是：
>
> 1. 任何类型的对象都可以放入其中；
> 2. 每次获取一个值后都必须要进行强制类型转换。

## 2. 定义简单泛型类

```
@ToString
@AllArgsConstructor
public class Pair<T> {
    private T first;
    private T second;
    
    public static void main(String[] args) {
        Pair<String> pair1 = new Pair<String>("firse", "second");
        System.out.println(pair1);
        
        Employee emp1 = new Employee("yan", "1310");
        Employee emp2 = new Employee("zhang", "1810");
        Pair<Employee> pair2 = new Pair<Employee>(emp1, emp2);
        System.out.println(pair2);
    }
}
```

> 在 Java 库中，使用变量 E 表示集合的元素类型，K 和 V 分别表示表的关键字和值的类型。T/U/S 表示任意类型。

## 3. 泛型方法

泛型方法 可以定义在普通类中，也可以定义在泛型类中。

```
public class Employee {
    public <T> T getInfo(T a) {
        return a;
    }
    public static void main(String[] args) {
        Employee emp = new Employee();
        String info1 = emp.<String>getInfo("I love you.");
        /*大多数情况下，方法调用中的 `<String>类型参数` 可以省略，编译器有足够的信息能够推断出所调用的方法。*/
        String info2 = emp.getInfo("I love you too.");
        System.out.println(info1);
        System.out.println(info2);
    }
}
```

## 4. 类型变量的限定

有时，类或方法需要对类型变量加以约束。例如，在方法中调用了 T 的 `compareTo` 方法，这时我们就得将 T 限制为实现了 `Comparable` 接口的类。

```
public static <T extends Comparable> T min(T[] a) {

}
```

`<T extends BoundingType>` 表示 T 应该是绑定类型的子类型（subtype）。

T 和 绑定类型可以是类，也可以是接口。 一个类型变量或通配符可以有多个限定，例如： `T extends Comparable & Serializable`。 限定类型用 & 分隔，用逗号来分隔类型变量。

## 5. 泛型代码和虚拟机

虚拟机没有泛型类型对象——所有对象都属于普通类。在泛型实现的早期版本中，甚至能够将使用泛型类型的程序编译为在1.0虚拟机上运行的类文件。这个向后兼容性在 Java 泛型开发的后期中放弃了。

### 5.1 类型擦除

在定义了一个泛型类型后，都会自动提供一个相应的`原始类型（raw type）`。原始类型的名字就是删去类型参数后的泛型类型名。

`擦除（erased）` 类型变量，并替换为限定类型（无限定的变量用 Object）。 例如， `Pair<T>` 的原始类型如下：

```
public class Pair {
    private Object first;
    private Object second;

    public Pair(Object first, Object second) {
        this.first = first;
        this.second = second;
    }

    public Object getFirst() {
        return first;
    }

    public void setFirst(Object first) {
        this.first = first;
    }

    public Object getSecond() {
        return second;
    }

    public void setSecond(Object second) {
        this.second = second;
    }
}
```

在程序中可以包含不同类型的 `Pair`，例如，`Pair<String>` 和 `Pair<LocalDate>`。但擦除类型后就变成了原始的 Pair 类型了。

原始类型用第一个限定的类型变量来替换，例如 `<T extends Comparable & Serializable>` 就会用 `Comparable` 类来替换。

### 5.2 翻译泛型表达式

当程序调用泛型方法时，如果擦除返回类型，编译器会插入强制类型转换。例如，

```
Pair<Employee> pair = ...;
Employee emp = pair.getFirst();
```

编辑器把这个方法调用翻译为两条虚拟机指令：

1. 对原始方法 `Pair.getFirst` 的调用；
2. 将返回的 `Object` 类型强制转换为 `Employee` 类型。

### 5.3 翻译泛型方法

类型擦除也会出现在泛型方法中。

方法的擦除会带来复杂的问题。举例：

```
public class DateInterval extends Pair<LocalDate> {

    public DateInterval(LocalDate first) {
        super(first, null);
    }

    @Override
    public void setSecond(LocalDate second) {
        if (second.isAfter(getFirst()))
            super.setSecond(second);
    }
}
```

上面这个类擦除后会变成：

```
public class DateInterval extends Pair {

    public DateInterval(LocalDate first) {
        super(first, null);
    }

    @Override
    public void setSecond(LocalDate second) {
        if (second.isAfter(getFirst()))
            super.setSecond(second);
    }
}
```

奇怪的是，该类还存在另一个从 `Pair` 继承的 `setSecond` 方法，即`public void setSecond(Object second)`。这显然是一个不同的方法，因为它有一个不同类型的参数——Object。

考虑下面的代码：

```
DateInterval interval = new DateInterval(LocalDate.of(1997, 10, 1));
Pair<LocalDate> pair = interval;
pair.setSecond(LocalDate.now());
```

希望对 `setSecond` 的调用有多态性，并调用最合适的那个方法。由于 Pair 引用 DateInterval 对象，所以应该调用 `DateInterval.setSecond`。问题在于类型擦除和多态发生了冲突。 要解决这个问题，就需要编译器在 `DateInterval` 类中生成一个桥方法(bridge method)。

```
public void setSecond(Object second) {
    setSecond((LocalDate)second);
}
```

Java 泛型转换的事实：

- 虚拟机中没有泛型，只有普通的类和方法。
- 所有的类型参数都用它们的限定类型替换。
- 桥方法被合成来保持多态。
- 为保持类型安全性，必要时插入强制类型转换。

### 5.4 约束与局限性

#### 5.4.1 不能用基本类型实例化类型参数

没有 `Pair<double>` ,只有 `Pair<Double>`。

#### 5.4.2 运行时类型查询只适用于原始类型

虚拟机中的对象总有一个特定的非泛型类型。因此，所有的类型查询只产生原始类型。

```
if(a instanceof Pair<String>)   //Error
if(a instanceof Pair<T>)   //Error
public static void main(String[] args) {
    Pair<String> pair1 = new Pair();
    Pair<Employee> pair2 = new Pair();
    if (pair1 instanceof Pair) {    //true
        System.out.println("true");
    }

    if (pair1.getClass() == pair2.getClass()) { //true
        System.out.println("they are equal;");
    }
}
```

#### 5.4.3 不能创建参数化类型的数组

例如:

```
Pair<String>[] pair3 = new Pair<>()[10];    //Error
```

#### 5.4.4 Varargs

问题：向参数个数可变的方法传递一个泛型类型的实例。

```
public static <T> void allAll(Collection<T> coll, T... ts) {
    for(t : ts) coll.add(t);
}
```

参数 ts 是一个数组，包含提供的所有实参。

对于下面的调用,尽管Java 虚拟机必须建立一个 Pair 数组，违法了前面的规则。但在这里规则有所放松，只会得到一个警告。

```
Collection<Pair<String>> table = ...;
Pair<String> pair1 = ...;
Pair<String> pair2 = ...;
allAll(table, pair1, pair2);
```

可以用两种方法抑制这种警告。

1. 为包含 addAll 调用的方法增加注解 `@SuppressWarnings("unchecked")`
2. 还可以直接在 allAll 方法上标注 `@SafeVarargs`。

#### 5.4.5 不能实例化类型变量

```java
new T();    //Error
```

类型擦除之后，T 就会变成 Object，肯定不希望调用 new Object()。

在 Java SE8之后，最好的解决方法是让调用者提供一个构造器方法。例如：

```java
Pair<String> p = Pair.makePair(String::new);
```

`makePair` 方法接口一个 `Supplier<T>`，这是一个函数式接口，表示一个无参树而且返回类型为 T 的函数。

```java
public static <T> Pair<T> makePair(Supplier<T> constr) {
    return new Pair<>(constr.get(), constr.get());
}
```

#### 5.4.6 不能构造泛型数组

#### 5.4.7 泛型类的静态上下文中类型变量无效

```java
private static T t; //Error
```

#### 5.4.8 不能抛出或捕获泛型类的实例

既不能抛出也不能捕获泛型类对象。实际上，甚至泛型类扩展 Throwable 都是不合法的。

```java
public class Problem<T> extends Exception {
    /* 不能正常编译 */
}
try {

}catch(T e) { //Errot---can't catch type variable

}
```

## 5.5 泛型类型的继承规则

在使用泛型类时，需要了解一些有关继承和子类型的准则。

对于一个类和一个子类，例如 `Employee` 和 `Manager`，`Pair<Employee>` 和 `Pair<Manager>`并没有什么关系。

泛型类可以扩展或实现其它的泛型类。例如 `ArrayList<E>` 实现 `List<E>`。这意味着 `ArrayList<Manager>` 可以转化为 `List<Manager>`，但是不能转化为 `ArrayList<Employee>` 或 `List<Employee>`。

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
        
}
```

## 5.6 通配符类型

固定的泛型类型系统使用起来有很多的限制。在 Java 中有一总巧妙地“解决方案”：通配符类型。

### 5.6.1 通配符概念

通配符类型 `Pair<? extends Employee>` 表示任何泛型 Pair 类型，它的类型参数是 Employee 的子类。

```java
public static void printInfo(Pair<? extends Employee> pair) {
    System.out.println(pair.getFirst() + " " + pair.getSecond());
}

public static void main(String[] args) {
    Manager manager1 = new Manager("yan", "111", 23, 20000, 30000);
    Manager manager2 = new Manager("zhang", "222", 25, 15674, 30000);
    Pair<Manager> pair = new Pair<>(manager1, manager2);
    Manager.printInfo(pair);
}
```

使用通配符会通过 `Pair<? extends Employee> pair` 的引用破坏 `Pair<Manager>` 吗？

```java
Pair<Manager> pair = new Pair<>(manager1, manager2);
Pair<? extends Employee> pair1 = pair;
Employee emp = new Employee();
pair1.setFirst(emp);    //编译错误
```

对 `setFirst` 的调用有一个类型错误。`Pair<? extends Employee>` 的方法似乎是这样的：

```java
? extends Employee getFirst()
void setFirst(? extends Employee)
```

这样将不可能调用 setFirst 方法。编译器只知道需要某个 Employee 的子类型，但不知道具体是什么类型。

### 5.6.2 通配符的超类型限定

可以指定一个超类型限定，例如： `? super Manager`，这个通配符限制为 `Manager` 的所有超类型。

# 六、集合

## 1、集合简介

在Java中，如果一个Java对象可以在内部持有若干其他Java对象，并对外提供访问接口，我们把这种Java对象称为集合。很显然，Java的数组可以看作是一种集合。

```java
String[] ss = new String[10]; // 可以持有10个String对象
ss[0] = "Hello"; // 可以放入String对象
String first = ss[0]; // 可以获取String对象
```

> 为什么在又了数组这种数据类型后，还需要其他集合类？
>
> 因为数组有如下的限制：
>
> - 数组初始化后的大小不能改变；
> - 数组只能按索引顺序存取。
>
> 因此，我们需要各种不同的集合来满足不同的需求。例如：
>
> - 可变大小的顺序链表；
> - 保证无重复元素的集合；
> - ...

集合主要包括 `Collection` 和 `Map` 两种，`Collection` 存储着对象的集合，而 `Map` 存储着键值对（两个对象）的映射表。

Java集合的设计有几个特点：

1. 实现了接口和实现类相分离，例如，有序表的接口是`List`，具体的实现类有`ArrayList`，`LinkedList`等；
2. 支持泛型，可以限制在一个集合中只放入同一种数据类型的元素。
3. 访问集合时总是通过统一的方式——迭代器（Iterator）来实现，它最明显的好处在于无需知道集合内部元素是按什么方式存储的。

> 由于Java的集合设计非常久远，中间经历过大规模改进，我们要注意到有一小部分集合类是遗留类，不应该继续使用：
>
> - `Hashtable`：一种线程安全的`Map`实现；
> - `Vector`：一种线程安全的`List`实现；
> - `Stack`：基于`Vector`实现的`LIFO`的栈。
>
> 还有一小部分接口是遗留接口，也不应该继续使用：
>
> - `Enumeration`：已被`Iterator`取代。

<h2>Collection</h2>

![image-20201123155855181](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201123155855181.png)

## 2、List

`List`是最基础的一种集合：它是一种有序链表。

`List` 的行为和数组几乎完全相同：`List` 内部按照放入元素的先后顺序存放，每个元素都可以通过索引值确定自己的位置。

### 2.1 ArrayList

在实际应用中，需要增删元素的有序列表，因此我们使用最多的是`ArrayList`。

实际上，`ArrayList` 在内部使用了数组来存储所有元素。它的处理和数组的使用类似，例如，一个ArrayList拥有5个元素，实际数组大小为`6`（即有一个空位）：

```ascii
size=5
┌───┬───┬───┬───┬───┬───┐
│ A │ B │ C │ D │ E │   │
└───┴───┴───┴───┴───┴───┘
```

当添加一个元素并指定索引到`ArrayList`时，`ArrayList`**自动移动**需要移动的元素：

```ascii
size=5
┌───┬───┬───┬───┬───┬───┐
│ A │ B │   │ C │ D │ E │
└───┴───┴───┴───┴───┴───┘
```

然后，往内部指定索引的数组位置添加一个元素，然后把`size`加`1`：

```ascii
size=6
┌───┬───┬───┬───┬───┬───┐
│ A │ B │ F │ C │ D │ E │
└───┴───┴───┴───┴───┴───┘
```

继续添加元素，但是数组已满，没有空闲位置的时候，`ArrayList`**先创建一个更大的新数组，然后把旧数组的所有元素复制到新数组**，紧接着用新数组取代旧数组：

```ascii
size=6
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ A │ B │ F │ C │ D │ E │   │   │   │   │   │   │
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
```

现在，新数组就有了空位，可以继续添加一个元素到数组末尾，同时`size`加`1`：

```ascii
size=7
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ A │ B │ F │ C │ D │ E │ G │   │   │   │   │   │
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
```

### 2.2 LinkedList

`LinkedList`通过“链表”也实现了List接口。在`LinkedList`中，它的内部每个元素都指向下一个元素：

```ascii
        ┌───┬───┐   ┌───┬───┐   ┌───┬───┐   ┌───┬───┐
HEAD ──>│ A │ ●─┼──>│ B │ ●─┼──>│ C │ ●─┼──>│ D │   │
        └───┴───┘   └───┴───┘   └───┴───┘   └───┴───┘
```

### 2.3 区别

|                     | ArrayList    | LinkedList           |
| :------------------ | :----------- | -------------------- |
| 获取指定元素        | 速度很快     | 需要从头开始查找元素 |
| 添加元素到末尾      | 速度很快     | 速度很快             |
| 在指定位置添加/删除 | 需要移动元素 | 不需要移动元素       |
| 内存占用            | 少           | 较大                 |

> 通常情况下，我们总是优先使用`ArrayList`。

### 2.4 遍历List

```java
   @Test
    public void m0() {
        List<String> list = new ArrayList<>();
        list.add("apple");
        list.add("pear");
        list.add("orange");

        //方法一：通过索引遍历，不推荐
        for (int i = 0; i < list.size(); ++i) {
            String str = list.get(i);
            System.out.println(str);
        }

        //方法二：通过Iterator遍历
        Iterator<String> iterator = list.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }

        //方法三：for earch方法遍历
        for (String str : list) {
            System.out.println(str);
        }

    }
```

上面通过三种方法遍历了List：

- 第一种方法并不推荐，一是因为代码复杂，二是因为`get(i)` 方法只有`ArrayList`的实现是高效的，换成`LinkedList`后，索引越大，访问速度越慢。
- 第二种方法：使用迭代器`Iterator`来访问`List`。`Iterator`本身也是一个对象，但它是由`List`的实例调用`iterator()`方法的时候创建的。`Iterator`对象知道如何遍历一个`List`，并且不同的`List`类型，返回的`Iterator`对象实现也是不同的，但总是具有最高的访问效率。
- 第三种方法：Java编译器会自动把`for each`循环变成`Iterator`的调用。

> 需要记住，通过`Iterator`遍历`List`永远是最高效的方式。

### 2.5 List和Array转换

#### List -> Array



#### Array -> List

对于JDK 11之前的版本，可以使用`Arrays.asList(T...)`方法把数组转换成`List`。但返回的是一个只读 `List` 。

```java
@Test
public void m1() {
    Integer[] array = { 1, 2, 3 };
    List<Integer> list = Arrays.asList(array);
    for (int i : list) {
        System.out.println(i);
    }

    list.add(5);    //java.lang.UnsupportedOperationException
}
```

对于高版本的，可以通过`List.of(T...)`方法转换，返回的也是只读 `List` 。

```java
Integer[] array = { 1, 2, 3 };
List<Integer> list = List.of(array);
```

### 2.6 Vector

和 ArrayList 类似，但它是线程安全的。

### 2.7编写equals方法

在 `List` 中提供了有两个方法：

- `boolean contains(Object o)` 方法判断 `List` 是否包含某个指定元素；
- `int indexOf(Object o)` 方法返回某个元素的索引，如果元素不存在，返回 `-1` 。

下面是一个例子：

```java
		@Test
    public void m3() {

        List<String> list = new ArrayList<>();
        list.add("apple");
        list.add("pear");
        list.add("orange");
        System.out.println(list.contains(new String("apple")));	//true
        System.out.println(list.contains(new String("banana")));	//false
        System.out.println(list.indexOf(new String("apple")));	//0
        System.out.println(list.indexOf(new String("banana")));	//-1
    }
```

虽然传入的 `new String("apple")` 和`List` 中的 `apple` 是不同的实例。但是仍然得到了`true`。这是因为在 `List` 内部并不是通过 `==` 来判断两个元素是否相等，而是通过 `equals()` 方法判断两个元素是否相等。源码如下：

```java
/*ArrayList.java*/
public boolean contains(Object o) {
  return indexOf(o) >= 0;
}

public int indexOf(Object o) {
  if (o == null) {
    for (int i = 0; i < size; i++)
      if (elementData[i]==null)
        return i;
  } else {
    for (int i = 0; i < size; i++)
      if (o.equals(elementData[i]))
        return i;
  }
  return -1;
}
```

因此，要正确使用 `List` 的`contains()`、`indexOf()`这些方法，必须要覆写对象的`equals()`方法。

> 之所以放入`String`、`Integer`这些对象能够得到预期结果，是因为Java标准库定义的这些类已经正确实现了`equals()`方法。

例如，在 `Student` 类没有编写 `equals` 方法时，不能得到正确结果。

```java
@Test
public void m3() {
  List<Student> students = new ArrayList<>();

  Student stu1 = new Student("zhangsan");
  students.add(stu1);

  Student stu2 = new Student("lisi");
  students.add(stu2);

  Student stu3 = new Student("wangwu");
  students.add(stu3);

  System.out.println(students.contains(new Student("zhangsan")));	//false
}
```

> 编写`equals()` 方法，必须满足的条件：
>
> - 自反性（Reflexive）：对于非`null`的`x`来说，`x.equals(x)`必须返回`true`；
> - 对称性（Symmetric）：对于非`null`的`x`和`y`来说，如果`x.equals(y)`为`true`，则`y.equals(x)`也必须为`true`；
> - 传递性（Transitive）：对于非`null`的`x`、`y`和`z`来说，如果`x.equals(y)`为`true`，`y.equals(z)`也为`true`，那么`x.equals(z)`也必须为`true`；
> - 一致性（Consistent）：对于非`null`的`x`和`y`来说，只要`x`和`y`状态不变，则`x.equals(y)`总是一致地返回`true`或者`false`；
> - 对`null`的比较：即`x.equals(null)`永远返回`false`。

根据规则，编写Student类的 `equals()` 方法：

```java
@Override
public boolean equals(Object obj) {
  if (obj instanceof Student) {
    Student student = (Student) obj;
    return Objects.equals(this.name, student.name) && this.id == student.id;
  }

  return false;
}
```

再来运行上一个测试代码，便能得到正确结果。

> 使用`Objects.equals()`比较两个引用类型是否相等的目的是省去了判断`null`的麻烦。两个引用类型都是`null`时也返回 `true`。

## 3、Set

如果我们只需要存储不重复的key，并不需要存储映射的value，那么就可以使用`Set`。在应该中，我们常用`Set` 来去除重复元素。

`Set`用于存储不重复的元素集合，它主要提供以下几个方法：

- 将元素添加进`Set`：`boolean add(E e)`
- 将元素从`Set`删除：`boolean remove(Object e)`
- 判断是否包含元素：`boolean contains(Object e)`

```java
@Test
public void m5() {
  Set<String> set = new HashSet<>();
  System.out.println(set.add("abc")); // true
  System.out.println(set.add("xyz")); // true
  System.out.println(set.add("xyz")); // false，添加失败，因为元素已存在
  System.out.println(set.contains("xyz")); // true，元素存在
  System.out.println(set.contains("XYZ")); // false，元素不存在
  System.out.println(set.remove("hello")); // false，删除失败，因为元素不存在
  System.out.println(set.size()); // 2，一共两个元素
}
```

> `Set`实际上相当于只存储key、不存储value的`Map`。我们经常用`Set`用于去除重复元素。
>
> 因为放入`Set`的元素和`Map`的key类似，都要正确实现`equals()`和`hashCode()`方法，否则该元素无法正确地放入`Set`。
>
> 最常用的`Set`实现类是`HashSet`，实际上，`HashSet`仅仅是对`HashMap`的一个简单封装，它的核心代码如下：
>
> ```java
> public class HashSet<E>	implements Set<E>
> {
>  private transient HashMap<E,Object> map;
> 
>  // Dummy value to associate with an Object in the backing Map
>  private static final Object PRESENT = new Object();
> 
>  public boolean add(E e) {
>    	return map.put(e, PRESENT)==null;
>  }
> 
> 	public boolean contains(Object o) {
>      return map.containsKey(o);
>  }
> 
>  public boolean remove(Object o) {
>      return map.remove(o) == PRESENT;
>  }
> }
> ```

### 3.1 TreeSet

`TreeSet` 类是一个有序集合(`sorted collection`)。可以以任意顺序将元素插入到集合中，在对集合进行遍历时，每个值将自动地按照排序后的顺序呈现。

正如 `TreeSet` 类名所示，排序使用树结构完成的（当前实现使用的是`红黑树`）。每次将一个元素添加到树中时，都被放置在正确的排序位置上。因此，迭代器总能以排好序的顺序访问每个元素。

将一个元素添加到树中要比添加到散列表中慢，但是与检查或链表中的重复元素相比还是快很多。如果树中包含n个元素，查找新元素的正确位置平均需要 $log_2n$ 次的比较。

> 要使用 `TreeSet`，必须能够比较元素。这些元素必须实现 `Comparable` 接口，或提供一个 `Comparator`。

### 3.2 HashSet

基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的。

### 3.3 LinkedHashSet

具有 HashSet 的查找效率，并且内部使用双向链表维护元素的插入顺序。

## 4、Queue

队列（`Queue`）是一种经常使用的集合。`Queue`实际上是实现了一个先进先出（FIFO：First In First Out）的有序表。它和 `List` 的区别就是：限制了添加和取出的方向。`Queue` 只能从末尾添加元素、从头部取出元素。

在Java的标准库中，队列接口`Queue`定义了以下几个方法：

- `int size()`：获取队列长度；
- `boolean add(E)`/`boolean offer(E)`：添加元素到队尾；
- `E remove()`/`E poll()`：获取队首元素并从队列中删除；
- `E element()`/`E peek()`：获取队首元素但并不从队列中删除。

对于具体的实现类，有的Queue有最大队列长度限制，有的Queue没有。

注意到添加、删除和获取队列元素总是有两个方法，这是因为在添加或获取元素失败时，这两个方法的行为是不同的。我们用一个表格总结如下：

|                    | throw Exception | 返回false或null    |
| :----------------- | :-------------- | :----------------- |
| 添加元素到队尾     | add(E e)        | boolean offer(E e) |
| 取队首元素并删除   | E remove()      | E poll()           |
| 取队首元素但不删除 | E element()     | E peek()           |

> 注意：不要把`null`添加到队列中，否则`poll()`方法返回`null`时，很难确定是取到了`null`元素还是队列为空。

### 4.1 LinkedList

`LinkedList`类即实现了`List`接口，又实现了`Queue`接口，但是，在使用的时候，如果我们把它当作List，就获取List的引用，如果我们把它当作Queue，就获取Queue的引用：

```
// 这是一个List:
List<String> list = new LinkedList<>();
// 这是一个Queue:
Queue<String> queue = new LinkedList<>();
```

### 4.2 PriorityQueue

`PriorityQueue` 是一种允许插队的 `Queue` 。它的出队顺序与元素的优先级有关。对`PriorityQueue`调用`remove()`或`poll()`方法，返回的总是优先级最高的元素。

元素的优先级是通过排序得到的，因此和`TreeMap` 、`TreeSet` 一样，放入`PriorityQueue`的元素，必须实现`Comparable`接口，如果没有实现，那么创建`PriorityQueue` 时必须传入一个`Comparator`对象。

```java
//方法一
@Test
public void m6() {
  Queue<Student> queue = new PriorityQueue<>();
  queue.add(new Student("zhangsan"));
  queue.add(new Student("lisi"));
  queue.add(new Student("asan"));
  queue.add(new Student("wangwu"));
  System.out.println(queue.remove());	//asan
  System.out.println(queue.remove());	//lisi
  System.out.println(queue.remove());	//wangwu
  System.out.println(queue.remove());	//zhangsan
}
```

## 5、Deque

`Deque` 是对 `Queue` 的一种变体，它允许两端都进，两端都出，叫做双端队列（Double Ended Queue）

`Queue`和`Deque`出队和入队的方法：

|                    | Queue                  | Deque                           |
| :----------------- | :--------------------- | :------------------------------ |
| 添加元素到队尾     | add(E e) / offer(E e)  | addLast(E e) / offerLast(E e)   |
| 取队首元素并删除   | E remove() / E poll()  | E removeFirst() / E pollFirst() |
| 取队首元素但不删除 | E element() / E peek() | E getFirst() / E peekFirst()    |
| 添加元素到队首     | 无                     | addFirst(E e) / offerFirst(E e) |
| 取队尾元素并删除   | 无                     | E removeLast() / E pollLast()   |
| 取队尾元素但不删除 | 无                     | E getLast() / E peekLast()      |

虽然`Deque` 是`Queue` 的扩展，`Queue` 提供的`add()`/`offer()`方法也可以使用，但是使用`Deque`时，最好不要调用`offer()`，而是调用`offerLast()`。



`Deque`是一个接口，它的实现类有`ArrayDeque`和`LinkedList`。

> `LinkedList`是一个全能选手，它即是`List`，又是`Queue`，还是`Deque`。但是我们在使用的时候，总是要用特定的接口来引用它。

## 6、Stack

栈（Stack）是一种后进先出（LIFO：Last In First Out）的数据结构。

`Stack`的操作：

- 把元素压栈：`push(E)`；
- 把栈顶的元素“弹出”：`pop(E)`；
- 取栈顶元素但不弹出：`peek(E)`。

> 在Java中，没有单独的 `Stack` 接口。

## 7、Collections

`Collections`是JDK提供的工具类，同样位于`java.util`包中。它提供了一系列静态方法，能更方便地操作各种集合。

> 注意Collections结尾多了一个s，不是Collection！

### 7.1 创建空集合

`Collections`提供了一系列方法来创建空集合：

- 创建空List：`List emptyList()`
- 创建空Map：`Map emptyMap()`
- 创建空Set：`Set emptySet()`

但是返回的空集合时不可变集合，无法向其中添加或删除元素。

```java
@Test
public void m8() {
  List<String> list = Collections.emptyList();
  list.add("111");	//java.lang.UnsupportedOperationException
}

```

### 7.2 创建单元素集合

`Collections`提供了一系列方法来创建一个单元素集合：

- 创建一个元素的List：`List singletonList(T o)`
- 创建一个元素的Map：`Map singletonMap(K key, V value)`
- 创建一个元素的Set：`Set singleton(T o)`

同样，返回的单元素集合也是不可变集合，无法向其中添加或删除元素。

### 7.3 排序

`Collections`可以对`List`进行排序。因为排序会直接修改`List`元素的位置，因此必须传入可变`List`：

```java
@Test
public void m8() {
  List<String> list = new ArrayList<>();
  list.add("apple");
  list.add("pear");
  list.add("orange");
  // 排序前:
  System.out.println(list);
  Collections.sort(list);
  // 排序后:
  System.out.println(list);
}
```

```java
//输出结果
[apple, pear, orange]
[apple, orange, pear]
```

### 7.4 洗牌

`Collections`提供了洗牌算法，即传入一个有序的`List`，可以随机打乱`List`内部元素的顺序，效果相当于让计算机洗牌：

```java
@Test
public void m8() {
  List<Integer> list = new ArrayList<>();
  for (int i=0; i<10; i++) {
    list.add(i);
  }
  // 洗牌前:
  System.out.println(list);
  Collections.shuffle(list);
  // 洗牌后:
  System.out.println(list);
}
```

```java
//输出结果
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
[5, 1, 4, 9, 2, 0, 7, 6, 8, 3]
```

### 7.5 不可变集合

`Collections`还提供了一组方法把可变集合封装成不可变集合：

- 封装成不可变List：`List unmodifiableList(List list)`
- 封装成不可变Set：`Set unmodifiableSet(Set set)`
- 封装成不可变Map：`Map unmodifiableMap(Map m)`

这种封装实际上是通过创建一个代理对象，拦截掉所有修改的方法。

```java
@Test
public void m9() {
  List<String> mutable = new ArrayList<>();
  mutable.add("apple");
  mutable.add("pear");
  // 变为不可变集合:
  List<String> immutable = Collections.unmodifiableList(mutable);
  immutable.add("orange"); // UnsupportedOperationException
}
```

但是继续对原始的可变`List`进行增删是可以的，并且，还会影响到封装后的“不可变”`List`：

```java
@Test
public void m8() {
  List<String> mutable = new ArrayList<>();
  mutable.add("apple");
  mutable.add("pear");
  // 变为不可变集合:
  List<String> immutable = Collections.unmodifiableList(mutable);
  mutable.add("orange");
  System.out.println(immutable);	//[apple, pear, orange]
}
```

所以在返回不可变 `List` 后，最好立刻将可变`List` 等于 `null` ，如下：

```java
@Test
public void m8() {
  List<String> mutable = new ArrayList<>();
  mutable.add("apple");
  mutable.add("pear");
  // 变为不可变集合:
  List<String> immutable = Collections.unmodifiableList(mutable);
  // 立刻扔掉mutable的引用:
  mutable = null;
  System.out.println(immutable);
}
```


<h2>Map</h2>

![image-20201123160601094](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201123160601094.png)

## 8、Map

`Map`这种键值（key-value）映射表的数据结构，作用是能高效地通过`key`快速查找`value`（元素）。比如，通过`name` 查询某个 `Student` 。

`Map` 和 `List` 一样，也是一个接口，常用的方法包括：

- `put(K key, V value)`方法：把`key`和`value`做了映射并放入`Map`
- `V get(K key)`：通过`key`获取到对应的`value`。如果`key`不存在，则返回`null`。
- `boolean containsKey(K key)`：查询某个 `key` 是否存在。

对于 `Map` 来说，常用的实现类是 `HashMap` 。

### 8.1 HashMap

#### 8.1.1 存储机制

1.8版本之前，HashMap采用**数组+链表**组成，数组用来映射hash，链表用来解决hash冲突。

1.8版本之后，HasHMap的结构有了改进，当**数组长度大于64，且链表长度大于8**，那么链表将会转化为**红黑树**。

当红黑树节点小于6之后，那么它将**退化为链表**。

[Java 8系列之重新认识HashMap](https://zhuanlan.zhihu.com/p/21673805)

![image-20201126111217853](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201126111217853.png)

#### 8.1.2 红黑树

<h4>性质</h4>

1. 每个节点不是红色就是黑色；
2. 不可能有连在一起的红色节点；
3. 根节点都是黑色；
4. 每个红色节点的两个子节点都是黑色，叶子结点【出度为0】都是黑色。

满足了这些性质就可以近似平衡了。

<h4>变换规则</h4>

旋转和颜色变换规则：==所有插入的点默认为红色==

<h5>变颜色</h5>

变颜色的情况：当前节点的父亲是红色，且它的祖父节点的另一个子节点【叔叔节点】也是红色。

1. 把父节点设为黑色；
2. 把叔叔节点设为黑色；
3. 把祖父节点【父亲的父亲】设为红色；
4. 把祖父节点设为当前要操作的节点；

<h5>左旋</h5>

左旋的情况：当前父节点是红色，叔叔是黑色，且当前节点是右子树。左旋以父节点作为左旋。

<h5>右旋</h5>

右旋的情况：当前父节点是红色，叔叔是黑色，且当前的节点是左子树。右旋：

1. 把父节点变成黑色；
2. 把祖父节点变为红色；
3. 以祖父节点旋转；

![image-20201126202744723](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201126202744723.png)

![image-20201126202807084](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201126202807084.png)



### 8.2 HashTable

和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程同时写入 `HashTable` 不会导致数据不一致。它是遗留类，不应该去使用它，而是使用 `ConcurrentHashMap` 来支持线程安全，`ConcurrentHashMap` 的效率会更高，因为 `ConcurrentHashMap` 引入了分段锁。

#### 8.2.1 fail-fast 机制

所谓快速失败就是在并发集合中，其进行迭代操作时，若有其他线程对其进行结构性的修改，这时迭代器会立马感知到，并且立即抛出 `ConcurrentModificationException` 异常，而不是等到迭代完成之后才告诉你

### 8.3 TreeMap

`Map` 接口的一个实现`HashMap` 是一种以空间换时间的映射表，它的实现原理决定了内部的 key 是无序的，导致了在遍历时的输出顺序是不可预测的。

如果要实现对 `key` 进行排序，就需要用到 `SortedMap` 接口，它的实现类是 `TreeMap` 。

```ascii
       ┌───┐
       │Map│
       └───┘
         ▲
    ┌────┴─────┐
    │          │
┌───────┐ ┌─────────┐
│HashMap│ │SortedMap│
└───────┘ └─────────┘
               ▲
               │
          ┌─────────┐
          │ TreeMap │
          └─────────┘
```

要使用 `TreeMap` 实现key的排序，放入`Map` 中的`key` 必须实现 `Comparable` 接口。当然对于 `String` 、`Integer` 这些类来说，已经实现了 `Comparable` 接口，因此可以直接使用。但对于自定义类，比如`Student` 类，则需要自己实现。

如果作为Key的class没有实现`Comparable`接口，那么，必须在创建`TreeMap`时指定一个自定义排序算法。

实现的例子如下：

```java
//方法一
//修改 Student.java
public class Student implements Comparable<Student> {
  
    //...
  
    @Override
    public int compareTo(Student o) {
        if (Objects.equals(this.name, o.getName())) {
            return 0;
        }

        return this.getName().compareTo(o.getName());
    }
}

//测试方法
@Test
public void m5() {
  Map<Student, Integer> map = new TreeMap<>();

  map.put(new Student("zhangsan"), 111);
  map.put(new Student("lisi"), 222);
  map.put(new Student("asan"), 333);
  map.put(new Student("wangwu"), 444);

  for (Student stu : map.keySet()) {
    System.out.println(stu);
  }
}
```

```java
//方法二
@Test
public void m5() {
  Map<Student, Integer> map = new TreeMap<>(new Comparator<Student>() {
    @Override
    public int compare(Student o1, Student o2) {
      if(Objects.equals(o1.getName(), o2.getName())) {
        return 0;
      }
      return o1.getName().compareTo(o2.getName());
    }
  });

  map.put(new Student("zhangsan"), 111);
  map.put(new Student("lisi"), 222);
  map.put(new Student("asan"), 333);
  map.put(new Student("wangwu"), 444);

  for (Student stu : map.keySet()) {
    System.out.println(stu);
  }
}
```

#### 原理

`TreeMap`基于红黑树实现，它包含几个重要的变量：`root`、`size` 、`comparator`。

![image-20201224170514282](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201224170514282.png)

### 8.4 EnumMap

如果`Map`传入的 `key` 对象是 `enum` 类型，那么可以使用 `EnumMap` ，它在内部以一个非常紧凑的数组存储value，并且根据 `enum` 类型的key直接定位到内部数组的索引，并不需要计算`hashCode()`，不但效率最高，而且没有额外的空间浪费。 

我们以 `DayOfWeek` （import java.time.DayOfWeek;）枚举来做一个“翻译”功能：

```java
@Test
public void m4() {
  Map<DayOfWeek, String> map = new EnumMap<>(DayOfWeek.class);
  map.put(DayOfWeek.MONDAY, "星期一");
  map.put(DayOfWeek.TUESDAY, "星期二");
  map.put(DayOfWeek.WEDNESDAY, "星期三");
  map.put(DayOfWeek.THURSDAY, "星期四");
  map.put(DayOfWeek.FRIDAY, "星期五");
  map.put(DayOfWeek.SATURDAY, "星期六");
  map.put(DayOfWeek.SUNDAY, "星期日");
  System.out.println(map);
  System.out.println(map.get(DayOfWeek.MONDAY));
}
```

### 8.5 LinkedHashMap

`LinkedHashSet` 和 `LinkedHashMap` 类用于记住插入元素项的顺序。 当条目插入到表中时，就会并入到双向链表中。

```java
@Test
public void testLinkedHashMap() {
    Map<String, Integer> map = new LinkedHashMap<>();
    map.put("a", 1);
    map.put("e", 4);
    map.put("c", 3);
    map.put("t", 9);
    
    Iterator<String> iterator = map.keySet().iterator();
    while (iterator.hasNext()) {
        String next = iterator.next();
        System.out.println(next);
    }   //a e c t

    Iterator<Integer> iterator1 = map.values().iterator();
    while (iterator1.hasNext()) {
        Integer next = iterator1.next();
        System.out.println(next);
    }   //1 4 3 9
}
```

默认情况下，链接散列映射是用插入顺序迭代，也可以设置 `LinkedHashMap` 为用访问顺序迭代。每次调用 get 或 put 之后，受影响的条目将从当前的位置删除，并放到链表的尾部（只有链表中的位置变化，散列表中的桶不会变化）。要构造这样一个散列映射表，需要使用 `public LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder)` 这个构造函数。

```java
@Test
public void testLinkedHashMap() {
    Map<String, Integer> map = new LinkedHashMap<String, Integer>(10, 0.75F, true);
    map.put("a", 1);
    map.put("e", 4);
    map.put("c", 3);
    map.put("t", 9);

    Integer e = map.get("e");
    
    Iterator<String> iterator = map.keySet().iterator();
    while (iterator.hasNext()) {
        String next = iterator.next();
        System.out.println(next);
    }   //a c t e
}
```

#### 原理

`LinkedHashMap`在传统的 `HashMap` 基础之上使用双向链表将每个节点连接在了一起：

![image-20201224171204814](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201224171204814.png)

![image-20201224174406743](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201224174406743.png)



事实上，`LinkedHashMap` 并不需要重写`put` 方法，在 `HashMap` 类中提供了如下3个 `callback`方法，`LinkedHashMap` 只需要实现这3个方法，`HashMap` 就会在 `put`、`get`和`remove`之后分别调用。

![image-20201224174618104](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201224174618104.png)

### 8.6 遍历Map

对 `Map` 来说，要遍历 `key` 可以使用 `for each` 循环遍历 `Map` 实例的 `keySet()` 方法返回的`Set`集合，它包含不重复的`key`的集合。

如果要遍历`key`和`value`，可以使用`for each`循环遍历`Map`对象的`entrySet()`集合，它包含每一个`key-value`映射。

具体如下：

```java
@Test
public void m3() {
  Map<String, Integer> map = new HashMap<>();
  map.put("apple", 123);
  map.put("pear", 456);
  map.put("banana", 789);

  //方法一
  for (String key : map.keySet()) {
    System.out.println(key + " " + map.get(key));
  }

  //方法二
  for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " " + entry.getValue());
  }
}
```

> `Map`和`List`不同：
>
> - `Map`存储的是`key-value`的映射关系，
> - 它*不保证顺序*。`Map` 的遍历顺序没有逻辑可言，甚至不同的JDK版本，相同的代码输出顺序都不同。

### 8.7 编写equals和hashCode

`HashMap` 之所以能够根据 `key` 直接拿到 `value` ，原因是它在内部通过空间换时间的方法，用一个大数组存储所有的 `value` ，并根据 `key` 直接计算出来 `value` 应该存储的位置。

```ascii
  ┌───┐
0 │   │
  ├───┤
1 │ ●─┼───> Student("Xiao Ming")
  ├───┤
2 │   │
  ├───┤
3 │   │
  ├───┤
4 │   │
  ├───┤
5 │ ●─┼───> Student("Xiao Hong")
  ├───┤
6 │ ●─┼───> Student("Xiao Jun")
  ├───┤
7 │   │
  └───┘
```

上述行为就是本科学的 `哈希表` ，根据 `key` 通过一个算法得到一个索引值，将 `value` 放到相应的位置。这个算法得到的索引值应该尽可能减少冲突，也就是说，对于两个 `key`：`a` 和`b` ，该算法得到的索引应该尽量不一样，如果发生冲突，也会有冲突解决办法。在 `HashMap` 中采用的办法是：如果发生冲突，在数组中，实际存储的就不是一个`Student` 实例，而是一个 `List` ，如下所示：

```ascii
  ┌───┐
0 │   │
  ├───┤
1 │   │
  ├───┤
2 │   │
  ├───┤
3 │   │
  ├───┤
4 │   │
  ├───┤
5 │ ●─┼───> List<Entry<String, Student>>
  ├───┤
6 │   │
  ├───┤
7 │   │
  └───┘
```

在查找时，先通过 `a` 得到索引5，再得到`List<Entry<String, Student>>`，接着它还需要遍历这个 `List` ，才能返回对应的`Student` 实例。所以如果冲突的概率越大，这个`List` 就越长，`Map` 的`get()` 方法查找效率就越低。

由上分析，得到结论：

- 作为 `key` 的类必须正确覆写 `equals()` 和`hashCode()`方法；
- 一个类如果覆写了`equals()`，就必须覆写`hashCode()`，并且覆写规则是：
  - 如果`equals()`返回`true`，则`hashCode()`返回值必须相等；
  - 如果`equals()`返回`false`，则`hashCode()`返回值尽量不要相等。
- 实现`hashCode()`方法可以通过`Objects.hashCode()`辅助方法实现

## 9、源码分析

[集合源码分析](https://www.cyc2018.xyz/Java/Java%20%E5%AE%B9%E5%99%A8.html#%E4%B8%89%E3%80%81%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90)

# 七、IO

IO是指Input/Output，这里的输入和输出是针对内存而言的：

- Input指从外部读入数据到内存，例如，把文件从磁盘读取到内存，从网络读取数据到内存等等。
- Output指把数据从内存输出到外部，例如，把数据从内存写入到文件，把数据从内存输出到网络等等。

> 为什么要把数据读到内存中才能处理这些数据呢？
>
> 是因为代码都是在内存中运行的，数据也必须读到内存，最终的表示方式无非是byte数组，字符串等，都必须存放在内存里。

从Java代码来看，输入实际上就是从外部，例如，硬盘上的某个文件，把内容读到内存，并且以Java提供的某种数据类型表示，例如，`byte[]`，`String`，这样，后续代码才能处理这些数据。又因为内存有“易失性”，所以必须要把处理后的数据输出保存。

I/O 流是一种顺序读写数据的模式，特点是单向流动。

`InputStream/OutputStream` 是以`byte`（字节）为最小单位单向流动；

`Reader/Writer` 是以 `char`（字符） 为最小单位单向流动。

<h2>同步和异步</h2>

**同步IO**是指，读写IO时代码必须等待数据返回后才继续执行后续代码，它的优点是代码编写简单，缺点是CPU执行效率低。

而**异步IO**是指，读写IO时仅发出请求，然后立刻执行后续代码，它的优点是CPU执行效率高，缺点是代码编写复杂。

Java标准库的包`java.io`提供了同步IO，而`java.nio`则是异步IO。上面我们讨论的`InputStream`、`OutputStream`、`Reader`和`Writer`都是同步IO的抽象类。

## 1、File

文件是一种十分重要的存储方式。在Java的标准库`java.io`中提供了`File` 对象来操作文件和目录。

构造一个 `File` 对象，需要传入一个文件路径。

```java
File f = new File("C:\\Windows\\notepad.exe");
```

> ps：
>
> 1. 构造File对象时，既可以传入绝对路径，也可以传入相对路径。
>
> 2. 在Java字符串中需要用`\\`表示Windows的分隔符`\`。Linux平台使用`/`作为路径分隔符
>
> 3. 可以用`.` 表示当前目录，`..` 表示上级目录。例如：
>
>    ```java
>    // 假设当前目录是C:\Docs
>    File f1 = new File("sub\\javac"); // 绝对路径是C:\Docs\sub\javac
>    File f3 = new File(".\\sub\\javac"); // 绝对路径是C:\Docs\sub\javac
>    File f3 = new File("..\\sub\\javac"); // 绝对路径是C:\sub\javac
>    ```
>
> 4. File对象有一个静态变量用于表示当前平台的系统分隔符：
>
>    ```java
>    System.out.println(File.separator); // 根据当前平台打印"\"或"/"
>    ```
>
>    

### 1.1 文件和目录

`isFile()` ：判断该`File` 对象是否是一个已存在的文件；

`isDirectory()` ：判断该 `File` 对象是否是一个已存在的目录；

用 `File` 对象获取到一个文件时，还可以判断文件的权限和大小：

- `boolean canRead()`：是否可读；
- `boolean canWrite()`：是否可写；
- `boolean canExecute()`：是否可执行；
- `long length()`：文件字节大小。

> 对于目录而言，不能直接通过 `length()` 方法获取大小。

### 1.2 创建和删除文件

当File对象表示一个文件时，可以通过`createNewFile()`创建一个新文件，用`delete()`删除该文件：

```java
@Test
public void m0() throws IOException {
  File file = new File("./aaa.txt");
  if (!file.exists()) {
    if (file.createNewFile()) {
      System.out.println("创建文件成功");
      if (file.delete()) {
        System.out.println("文件已删除");
      }
    }
  }

}
```

有些时候，程序需要读写一些临时文件，File对象提供了`createTempFile()`来创建一个临时文件，以及`deleteOnExit()`在JVM退出时自动删除该文件。

```java
public class Main {
    public static void main(String[] args) throws IOException {
        File f = File.createTempFile("tmp-", ".txt"); // 提供临时文件的前缀和后缀
        f.deleteOnExit(); // JVM退出时自动删除
        System.out.println(f.isFile());
        System.out.println(f.getAbsolutePath());
    }
}
```

### 1.3 遍历文件和目录

当File对象表示一个目录时，可以使用 `list()` 和 `listFiles()` 列出目录下的文件和目录。两者的区别如下：

- `public String[] list()` ：返回的是一个字符串数组，内容是`File` 对象表示目录中的文件名和目录名，并不是完整的路径。
- `public File[] listFiles()`：返回的是一个 `File` 数组，表示该目录下的文件和目录名，可以通过 `gtePath()` 获得路径。

`list()` 和 `listFiles()` 还提供了一系列的重载方法，可以过滤不想要的文件和目录。

```java
@Test 
public void m1() {
  File file = new File("/Users/zhang/Desktop/2020春上课");
  String[] list = file.list();
  for (String str : list) {
    System.out.println(str);
  }

  System.out.println("========================");
  File[] files = file.listFiles();
  for (File f : files) {
    System.out.println(f.getName());
  }
  
  System.out.println("========================");

  File[] files1 = file.listFiles(new FilenameFilter() { // 仅列出.png文件
    public boolean accept(File dir, String name) {
      return name.endsWith(".png"); // 返回true表示接受该文件
    }
  });
  for (File f : files1) {
    System.out.println(f.getPath());
  }

}
```

和文件操作类似，File对象如果表示一个目录，可以通过以下方法创建和删除目录：

- `boolean mkdir()`：创建当前File对象表示的目录；
- `boolean mkdirs()`：创建当前File对象表示的目录，并在必要时将不存在的父目录也创建出来；
- `boolean delete()`：删除当前File对象表示的目录，当前目录必须为空才能删除成功。

## 2、InputStream

`InputStream`是Java标准库提供的最基本的输入流。它位于`java.io`这个包里。`java.io`包提供了所有同步IO的功能。

`InputStream`只是一个抽象类，它是所有输入流的超类。

### 2.1 FileInputStream

`FileInputStream`是`InputStream`的一个子类。它是从文件流中读取数据。下面是读取 `FileInputStream` 所有字节的例子：

```java
@Test
public void m2() throws IOException {
    try (InputStream inputStream = new FileInputStream("/Users/zhangsan/Desktop/111.png")) {
        byte[] bytes = new byte[1024];
        int n = 0;
        while ((n = inputStream.read(bytes)) != -1) {
            System.out.println(bytes);
        }
    }
}
```

> 上面的 `try(resource)` 的语法，是和在`finally` 中调用 `close()`一样的效果  ，是让编译器自动为我们关闭资源。编译器是看 `try(resource = ...)` 中的对象是否实现了`java.lang.AutoCloseable`接口，如果实现了，就自动加上`finally`语句并调用`close()`方法。

### 2.2 ByteArrayInputStream

`ByteArrayInputStream` 可以在内存中模拟一个`InputStream`，虽然在实际应该中并不多，但在测试时，可以使用它来构造一个 `InputStream` ，不用真实的创建一个文件来读取。

```java
@Test
public void m2() throws IOException {
  byte[] data = { 72, 101, 108, 108, 111, 33 };
  try (InputStream input = new ByteArrayInputStream(data)) {
    int n;
    while ((n = input.read()) != -1) {
      System.out.println(n);
    }
  }
}
```

## 3、OutputStream

`OutputStream`是Java标准库提供的最基本的输出流。

`OutputStream`也是抽象类，它是所有输出流的超类。

需要特别注意的是：`OutputStream` 还提供了一个 `flush()` 方法，它的目的是将缓冲区的内容真正输出到目的地。

> 为什么要有 `flush()` 方法？
>
> 因为向磁盘、网络写入数据的时候，出于效率的考虑，操作系统并不是输出一个字节就立刻写入到文件或者发送到网络，而是把输出的字节先放到内存的一个缓冲区里（本质上就是一个`byte[]`数组），等到缓冲区写满了，再一次性写入文件或者网络。对于很多IO设备来说，一次写一个字节和一次写1000个字节，花费的时间几乎是完全一样的，所以`OutputStream`有个`flush()`方法，能强制把缓冲区内容输出。

`OutputStream` 会在缓冲区满了之后，自动调用 `flush()` ，并且，在调用`close()`关闭`OutputStream`之前，也会自动调用`flush()`方法。但是如果我们想要把缓冲区中的内容立刻发送出去，就必须自己调用 `flush()` 。

### 3.1 FileOutputStream

```java
@Test
public void m2() throws IOException {
  try (OutputStream output = new FileOutputStream("./readme.txt")) {
    output.write("Hello".getBytes("UTF-8")); // Hello
  } // 编译器在此自动为我们写入finally并调用close()
}
```

### 3.2 ByteArrayOutputStream

`ByteArrayOutputStream`可以在内存中模拟一个`OutputStream`：

```java
@Test
public void m2() throws IOException {
  byte[] data;
  try (ByteArrayOutputStream output = new ByteArrayOutputStream()) {
    output.write("Hello ".getBytes("UTF-8"));
    output.write("world!".getBytes("UTF-8"));
    data = output.toByteArray();
  }
  System.out.println(new String(data, "UTF-8"));
}
```



## 4、Filter模式

在Java中，JDK将`InputStream`分为两大类：

一类是直接提供数据的基础`InputStream`，例如：

- `FileInputStream`
- `ByteArrayInputStream`
- `ServletInputStream`
- ...

一类是提供额外附加功能的`InputStream`，例如：

- `BufferedInputStream`
- `DigestInputStream`
- `CipherInputStream`
- ...

当我们想给一个“基础”`InputStream` ，例如`FileInpuStream`，提供缓冲的功能时，可以用`BufferedInputStream`包装这个`InputStream`，得到的包装类型是`BufferedInputStream`，但它仍然被视为一个`InputStream`：

```java
InputStream buffered = new BufferedInputStream(file);
```

如果还想要其他附加功能，可以继续包装。无论我们包装多少次，得到的对象始终是 `InputStream`。

 上述这种通过一个“基础”组件再叠加各种“附加”功能组件的模式，称之为Filter模式（或者装饰器模式：Decorator）

## 5、操作Zip

`ZipInputStream`是一种`FilterInputStream`，它可以直接读取zip包的内容：

```ascii
┌───────────────────┐
│    InputStream    │	->java.io
└───────────────────┘
          ▲
          │
┌───────────────────┐
│ FilterInputStream │	->java.io
└───────────────────┘
          ▲
          │
┌───────────────────┐
│InflaterInputStream│	->java.util.zip
└───────────────────┘
          ▲
          │
┌───────────────────┐
│  ZipInputStream   │	->java.util.zip
└───────────────────┘
          ▲
          │
┌───────────────────┐
│  JarInputStream   │	->java.util.jar
└───────────────────┘
```

[压缩和解压](https://blog.csdn.net/weixin_40971059/article/details/104775323)

## 6、Reader

`Reader`是Java的IO库提供的另一个输入流接口。和`InputStream`的区别是，`InputStream`是一个字节流，即以`byte`为单位读取，而`Reader`是一个字符流，即以`char`为单位读取：

| InputStream                         | Reader                                |
| :---------------------------------- | :------------------------------------ |
| 字节流，以`byte`为单位              | 字符流，以`char`为单位                |
| 读取字节（-1，0~255）：`int read()` | 读取字符（-1，0~65535）：`int read()` |
| 读到字节数组：`int read(byte[] b)`  | 读到字符数组：`int read(char[] c)`    |

`java.io.Reader` 是所有字符输入流的超类。

### 6.1 FileReader

`FileReader`是`Reader`的一个子类，它可以打开文件并获取`Reader`。下面是如何读取一个 `FileReader` 的所有字符：

 ```java
/*缓冲区的方法*/
public void readFile() throws IOException {
    try (Reader reader = new FileReader("src/readme.txt", StandardCharsets.UTF_8)) {
        char[] buffer = new char[1000];	//设置缓冲区
        int n;
        while ((n = reader.read(buffer)) != -1) {
            System.out.println("read " + n + " chars.");
        }
    }
}
 ```

> 如果文件中包含中文，可能会乱码，因为`FileReader`默认的编码与系统相关，例如，Windows系统的默认编码可能是`GBK`，打开一个`UTF-8`编码的文本文件就会出现乱码。要避免这样的问题，需要在创建 `FileReader` 时指定编码。
>
> ```java
> Reader reader = new FileReader("src/readme.txt", StandardCharsets.UTF_8);
> ```



### 6.2 CharArrayReader

和`ByteArrayInputStream`类似，`CharArrayReader`可以在内存中模拟一个`Reader`，它的作用实际上是把一个`char[]`数组变成一个`Reader`：

```java
try (Reader reader = new CharArrayReader("Hello".toCharArray())) {
}
```

### 6.3 StringReader

`StringReader` 和`CharArrayReader` 几乎一样，只是把 `String` 作为数据源：

```java
try (Reader reader = new StringReader("Hello")) {
}
```

### 6.4 Reader 和 InputStream关系

除了特殊的`CharArrayReader`和`StringReader`，普通的`Reader`实际上是基于`InputStream`构造的，因为`Reader`需要从`InputStream`中读入字节流（`byte`），然后，根据编码设置，再转换为`char`就可以实现字符流。

### 6.5 InputStreamReader

根据上面的解释可以知道，把一个`InputStream`转换为`Reader`是完全可行的。`InputStreamReader`就是这样一个转换器，它可以把任何`InputStream`转换为`Reader`。

```java
// 持有InputStream:
InputStream input = new FileInputStream("src/readme.txt");
// 变换为Reader:
Reader reader = new InputStreamReader(input, "UTF-8");
```

简洁的写法：

```java
try (Reader reader = new InputStreamReader(new FileInputStream("src/readme.txt"), "UTF-8")) {
    // TODO:
}
```

## 7、Writer

`Reader`是带编码转换器的`InputStream`，它把`byte`转换为`char`，类似的`Writer`就是带编码转换器的`OutputStream`，它把`char`转换为`byte`并输出。

`Writer`和`OutputStream`的区别如下：

| OutputStream                           | Writer                                   |
| :------------------------------------- | :--------------------------------------- |
| 字节流，以`byte`为单位                 | 字符流，以`char`为单位                   |
| 写入字节（0~255）：`void write(int b)` | 写入字符（0~65535）：`void write(int c)` |
| 写入字节数组：`void write(byte[] b)`   | 写入字符数组：`void write(char[] c)`     |
| 无对应方法                             | 写入String：`void write(String s)`       |

### 7.1 FileWriter

`FileWriter`就是向文件中写入字符流的`Writer`：

```java
try (Writer writer = new FileWriter("readme.txt", StandardCharsets.UTF_8)) {
    writer.write('H'); // 写入单个字符
    writer.write("Hello".toCharArray()); // 写入char[]
    writer.write("Hello"); // 写入String
}
```

### 7.2 CharArrayWriter

`CharArrayWriter`可以在内存中创建一个`Writer`，它的作用实际上是构造一个缓冲区，可以写入`char`，最后得到写入的`char[]`数组，这和`ByteArrayOutputStream`非常类似：

```java
try (CharArrayWriter writer = new CharArrayWriter()) {
    writer.write(65);
    writer.write(66);
    writer.write(67);
    char[] data = writer.toCharArray(); // { 'A', 'B', 'C' }
}
```

### 7.3 StringWriter

`StringWriter`也是一个基于内存的`Writer`，它和`CharArrayWriter`类似。实际上，`StringWriter`在内部维护了一个`StringBuffer`，并对外提供了`Writer`接口。

### 7.4 OutputStreamWriter

和 ` InputStreamReader`  类似，可以通过`OutputStreamWriter`将`OutputStream`转换为`Writer`，转换时需要指定编码：

```java
try (Writer writer = new OutputStreamWriter(new FileOutputStream("readme.txt"), "UTF-8")) {
    // TODO:
}
```

## 8、对象操作和序列化

为了保存对象数据，需要打开一个 ObjectOutputStream 对象：

```java
ObjectOutputStream outputStream = new ObjectOutputStream(new FileOutputStream("Employee.dat"));
Employee employee = new Employee();
outputStream.writeObject(employee);
```

如果想要把该对象读回，类似地，也需要一个 ObjectInputStream 对象。

但是对于 Employee 类来说，必须实现 Serializable 接口。
`public class Employee implements Serializable {...}`

> 不会对静态变量进行序列化，因为序列化只是保存对象的状态，静态变量属于类的状态。

### 8.2 序列化的作用

例如，下面这样的对象引用网络，两个 Manager 对象都包含一个表示秘书的 Employee 对象的引用。 在保存这样的关系时，当然不能保存 Employee 对象的内存地址，因为当对象重新加载时，内存地址就变化了。

![img](https://gitee.com/yanjundong97/picBed/raw/master/images/对象序列化1.png)

解决办法：每个对象都是用一个序列号来保存的。下面是其算法：

- 对遇到的每一个对象都关联一个序列号
- 如果遇到一个新的对象，就保存其对象到输出流中
- 如果某个对象之前已经保存过了，那么就只写出 “与之前保存过的序列号为x的对象相同”。 

![img](https://gitee.com/yanjundong97/picBed/raw/master/images/对象序列化1.png)

在读回对象时，整个过程是反过来的。

### 8.2 transient

`transient` 关键字可以使一些属性不会被序列化。

`ArrayList` 中存储数据的数组 `elementData` 是用 `transient` 修饰的，因为这个数组是动态扩展的，并不是所有的空间都被使用，因此就不需要所有的内容都被序列化。通过重写序列化和反序列化方法，使得可以只序列化数组中有内容的那部分数据。

```java
private transient Object[] elementData;
```

## 9、网络操作

Java 中的网络支持：

- `InetAddress`：用于表示网络上的硬件资源，即 IP 地址；
- `URL`：统一资源定位符；
- `Sockets`：使用 TCP 协议实现网络通信；
- `Datagram`：使用 UDP 协议实现网络通信。

## 10、BIO、NIO、AIO的理解

### 阻塞与非阻塞

#### 阻塞

![image-20210304094607128](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210304094607128.png)

阻塞IO情况下，当用户调用`read`后，用户线程会被阻塞，等内核数据准备好并且数据从内核缓冲区拷贝到用户态缓存区后`read`才会返回。存在两部分阻塞：

1. CPU把数据从磁盘读到内核缓冲区。
2. CPU把数据从内核缓冲区拷贝到用户缓冲区。

#### 非阻塞

![image-20210304100116634](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210304100116634.png)

非阻塞IO发出read请求后发现数据没准备好，会继续往下执行，此时应用程序会不断轮询询问数据是否准备好，当数据没有准备好时，内核立即返回 `EWOULDBLOCK`错误。直到数据被拷贝到应用程序缓冲区，read请求才获取到结果。

但**内核态的数据拷贝到用户程序的缓存区这个过程**仍然是阻塞的。

#### IO多路复用

![image-20210304100639041](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210304100639041.png)

单个线程通过记录跟踪每一个Sock(I/O流)的状态来同时管理多个I/O流. 发明它的目的是尽量多的提高服务器的吞吐能力。实现一个线程监控多个IO请求，哪个IO有请求就把数据从内核拷贝到进程缓冲区，拷贝期间是阻塞的！

I/O多路复用的具体的实现有`select`、`poll`、`epoll`。

##### `select`

select是第一版IO复用，提出后暴漏了很多问题。

1. select 会修改传入的参数数组，这个对于一个需要调用很多次的函数，是非常不友好的。
2. select 如果任何一个`sock(I/O stream)`出现了数据，select 仅仅会返回，但不会告诉是哪个sock上有数据，只能自己遍历查找。
3. select 只能监视1024个连接。
4. **select 不是线程安全的**，如果你把一个sock加入到select, 然后突然另外一个线程发现这个sock不用，要收回，这个select 不支持的。

##### `poll`

poll 修复了 select 的很多问题。

1. poll 去掉了1024个链接的限制。
2. poll 从设计上来说不再修改传入数组。

但是poll仍然不是线程安全的， 这就意味着不管服务器有多强悍，你也只能在一个线程里面处理一组 I/O 流。你当然可以拿多进程来配合了，不过然后你就有了多进程的各种问题。

##### `epoll`

epoll 可以说是 I/O  多路复用最新的一个实现，epoll 修复了poll 和select绝大部分问题， 比如：

1. epoll 现在**是线程安全的**。
2. epoll 现在不仅告诉你sock组里面数据，还会告诉你具体哪个sock有数据，你不用自己去找了。
3. epoll 内核态管理了各种IO文件描述符， 以前用户态发送所有文件描述符到内核态，然后内核态负责筛选返回可用数组，现在epoll模式下所有文件描述符在内核态有存，查询时不用传文件描述符进去了。

但epoll的缺点是只有Linux支持，比如平常Nginx为何可以支持4W的QPS是因为它会使用目标平台上面最高效的I/O多路复用模型。

### 同步与异步IO

同步跟异步的区别在于`数据从内核空间拷贝到用户空间是否由用户线程完成`。

#### 同步IO

同步IO又分为同步阻塞跟同步非阻塞两种。

![image-20210304101911162](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210304101911162.png)

#### 异步IO

![image-20210304101648306](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210304101648306.png)

### Java IO

#### BIO

**同步阻塞IO**，一个连接一个线程。

![image-20210304102416020](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210304102416020.png)

#### NIO

**同步非阻塞IO**

![image-20210304102635512](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210304102635512.png)

#### AIO

**异步非阻塞IO**，相比NIO更进一步，进程读取数据时只负责发送跟接收指令，数据的准备工作完全由操作系统来处理。

[阻塞、非阻塞、多路复用、同步、异步、BIO、NIO、AIO 一锅端](https://mp.weixin.qq.com/s/17S9Io1dIS_fihfLk5bsAQ)

# 八、日期和时间

Java标准库有两套处理日期和时间的API：

- 一套定义在`java.util`这个包里面，主要包括`Date`、`Calendar`和`TimeZone`这几个类；
- 一套新的API是在Java 8引入的，定义在`java.time`这个包里面，主要包括`LocalDateTime`、`ZonedDateTime`、`ZoneId`等。

在旧的API中存在有很多问题，所以引入了新的API。但很多时候，我们仍然需要在两种对象之间进行转换。

在使用日期和时间时，除非涉及到遗留代码，否则应该坚持使用新的API。

## 1、Date 和 Calendar

### 1.1 Date格式输出

```java
@Test
public void m0() {
  Date date = new Date();	//获取当前时间
  DateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");	//自定义格式输出
  System.out.println(sdf.format(date));
}
```

> Date中的 `getDate()`、 `getDay()` 、`getHours()` 、`getMinites()` 等方法已弃用。

### 1.2 Calendar

`Calendar`可以用于获取并设置年、月、日、时、分、秒，它和`Date`比，主要多了一个可以做简单的日期和时间运算的功能。基本用法如下：

```java
@Test
public void m0() {
  // 获取当前时间:
  Calendar c = Calendar.getInstance();
  int y = c.get(Calendar.YEAR);
  int m = 1 + c.get(Calendar.MONTH);
  int d = c.get(Calendar.DAY_OF_MONTH);
  int w = c.get(Calendar.DAY_OF_WEEK);
  int hh = c.get(Calendar.HOUR_OF_DAY);
  int mm = c.get(Calendar.MINUTE);
  int ss = c.get(Calendar.SECOND);
  int ms = c.get(Calendar.MILLISECOND);
  System.out.println(y + "-" + m + "-" + d + " " + w + " " + hh + ":" + mm + ":" + ss + "." + ms);
}
```

```java
//输出结果
2020-3-10 3 21:10:20.43
```

> `Calendar` 获取年月日这些信息变成了`get(int field)` ，返回的年份不必转换，返回的月份必须要加1。
>
> 返回的星期：，`1`~`7`分别表示周日，周一，……，周六。
>
> `Calendar` 只有一种获取方式，即`Calendar.getInstance()` 获取当地时间。

给`Calendar` 设置一个特定的日期和时间：

```java
@Test
public void m1() {
  // 当前时间:
  Calendar c = Calendar.getInstance();
  // 清除所有:
  c.clear();
  // 设置2019年:
  c.set(Calendar.YEAR, 2019);
  // 设置9月:注意8表示9月:
  c.set(Calendar.MONTH, 8);
  // 设置2日:
  c.set(Calendar.DATE, 2);
  // 设置时间:
  c.set(Calendar.HOUR_OF_DAY, 21);
  c.set(Calendar.MINUTE, 22);
  c.set(Calendar.SECOND, 23);
  //getTime()  可以将Calendar对象转换为Date对象
  String format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(c.getTime());
  System.out.println(format);	//2019-09-02 21:22:23
}
```

## 2、LocalDateTime

从Java 8开始，`java.time`包提供了新的日期和时间API，主要涉及的类型有：

- 本地日期和时间：`LocalDateTime`，`LocalDate`，`LocalTime`；
- 带时区的日期和时间：`ZonedDateTime`；
- 时刻：`Instant`；
- 时区：`ZoneId`，`ZoneOffset`；
- 时间间隔：`Duration`。

以及一套新的用于取代`SimpleDateFormat`的格式化类型`DateTimeFormatter`。

新的API：

- 严格区分了时刻、本地日期、本地时间和带失去的日期时间，并且对日期和时间的运算更加方便。
- Month用1～12表示1月到12月；Week用1～7表示周一到周日；
- 类型几乎都是不变类型（类似String），不必担心被修改。

### 2.1 LocalDateTime

```java
@Test
public void m1() {
  LocalDate d = LocalDate.now(); // 当前日期
  LocalTime t = LocalTime.now(); // 当前时间
  LocalDateTime dt = LocalDateTime.now(); // 当前日期和时间
  System.out.println(d); // 严格按照ISO 8601格式打印
  System.out.println(t); // 严格按照ISO 8601格式打印
  System.out.println(dt); // 严格按照ISO 8601格式打印
}
```

```java
//输出结果
2020-03-10
21:48:13.924
2020-03-10T21:48:13.924
```

### 2.2 日期和时间	<->	日期、时间

```java
LocalDateTime dt = LocalDateTime.now(); // 当前日期和时间
LocalDate d = dt.toLocalDate(); // 转换到当前日期
LocalTime t = dt.toLocalTime(); // 转换到当前时间
```

```java
// 指定日期和时间:
LocalDate d2 = LocalDate.of(2019, 11, 30); // 2019-11-30, 注意11=11月
LocalTime t2 = LocalTime.of(15, 16, 17); // 15:16:17
LocalDateTime dt2 = LocalDateTime.of(2019, 11, 30, 15, 16, 17);
LocalDateTime dt3 = LocalDateTime.of(d2, t2);
```

### 2.3 String ->LocalDateTime

因为严格按照ISO 8601的格式，所以可以把字符串转换为`LocalDateTime`：

```java
LocalDateTime dt = LocalDateTime.parse("2019-11-19T15:16:17");
LocalDate d = LocalDate.parse("2019-11-19");
LocalTime t = LocalTime.parse("15:16:17");
```

> 注意ISO 8601规定的日期和时间分隔符是`T`。标准格式如下：
>
> - 日期：yyyy-MM-dd
> - 时间：HH:mm:ss
> - 带毫秒的时间：HH:mm:ss.SSS
> - 日期和时间：yyyy-MM-dd'T'HH:mm:ss
> - 带毫秒的日期和时间：yyyy-MM-dd'T'HH:mm:ss.SSS

### 2.4 DateTimeFormater

如果要自定义输出的格式，或者要把一个非ISO 8601格式的字符串解析成`LocalDateTime`，可以使用新的`DateTimeFormatter`：

```java
@Test
public void m1() {
  // 自定义格式化:
  DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss");
  System.out.println(dtf.format(LocalDateTime.now()));

  // 用自定义格式解析:
  LocalDateTime dt2 = LocalDateTime.parse("2020/03/09 15:16:17", dtf);
  System.out.println(dt2);
}
```

```java
//输出结果
2020/03/10 22:02:57
2020-03-09T15:16:17
```

### 2.5 LocalDateTime 的计算

`LocalDateTime`提供了对日期和时间的加减计算方法：

```java
@Test
public void m1() {
  LocalDateTime dt = LocalDateTime.of(2020, 03, 10, 20, 30, 59);
  System.out.println(dt); //2020-03-10T20:30:59
  // 加5天减3小时:
  LocalDateTime dt2 = dt.plusDays(5).minusHours(3);
  System.out.println(dt2); // 2020-03-15T17:30:59
  // 减1月:
  LocalDateTime dt3 = dt2.minusMonths(1);
  System.out.println(dt3); // 2020-02-15T17:30:59
  // 日期变为15日:
  LocalDateTime dt4 = dt.withDayOfMonth(15);
  System.out.println(dt4); // 2020-03-15T20:30:59
  // 月份变为9:
  LocalDateTime dt5 = dt2.withMonth(9);
  System.out.println(dt5); // 2020-09-15T17:30:59
}
```

> 对日期和时间进行调整则使用`withXxx()`方法，例如：`withHour(15)`会把`10:11:12`变为`15:11:12`：
>
> - 调整年：withYear()
> - 调整月：withMonth()
> - 调整日：withDayOfMonth()
> - 调整时：withHour()
> - 调整分：withMinute()
> - 调整秒：withSecond()
>
> 调整月时，如果填入错误日期，例如32，会报错。其余类似。
>
> 调整月份时，会相应地调整日期，即把`2020-10-31`的月份调整为`9`时，日期也自动变为`30`。



### 2.6 `with()`方法

`LocalDateTime`还有一个通用的`with()`方法允许我们做更复杂的运算。例如：

```java
    @Test
    public void m1() {
        // 本月第一天0:00时刻:
        LocalDateTime firstDay = LocalDate.now().withDayOfMonth(1).atStartOfDay();
        System.out.println(firstDay);   //2020-03-01T00:00

        // 本月最后1天:
        LocalDate lastDay = LocalDate.now().with(TemporalAdjusters.lastDayOfMonth());
        System.out.println(lastDay);    //2020-03-31

        // 下月第1天:
        LocalDate nextMonthFirstDay = LocalDate.now().with(TemporalAdjusters.firstDayOfNextMonth());
        System.out.println(nextMonthFirstDay);  //2020-04-01

        // 本月第1个周一:
        LocalDate firstWeekday = LocalDate.now().with(TemporalAdjusters.firstInMonth(DayOfWeek.MONDAY));
        System.out.println(firstWeekday);   //2020-03-02
    }
```

### 2.7 日期时间先后判断

要判断两个`LocalDateTime`的先后，可以使用`isBefore()`、`isAfter()`方法，对于`LocalDate`和`LocalTime`类似：

```java
@Test
public void m1() {
  LocalDateTime now = LocalDateTime.now();
  LocalDateTime target = LocalDateTime.of(2019, 11, 19, 8, 15, 0);
  System.out.println(now.isBefore(target));
  System.out.println(LocalDate.now().isBefore(LocalDate.of(2019, 11, 19)));
  System.out.println(LocalTime.now().isAfter(LocalTime.parse("08:15:00")));
}
```

### 2.8 Duration和Period

`Duration`表示两个时刻之间的时间间隔。另一个类似的`Period`表示两个日期之间的天数：

```java
    @Test
    public void m1() {
        LocalDateTime start = LocalDateTime.of(2019, 11, 19, 8, 15, 0);
        LocalDateTime end = LocalDateTime.of(2020, 1, 9, 19, 25, 30);
        Duration d = Duration.between(start, end);
        System.out.println(d); // PT1235H10M30S

        Period p = LocalDate.of(2019, 11, 19).until(LocalDate.of(2020, 1, 9));
        System.out.println(p); // P1M21D
    }
```

> 两个`LocalDateTime`之间的差值使用`Duration`表示，类似`PT1235H10M30S`，表示1235小时10分钟30秒。
>
> 两个`LocalDate`之间的差值用`Period`表示，类似`P1M21D`，表示1个月21天。

## 3、ZonedDateTime

当要表示一个带时区的日期和时间时，就需要用到 `ZonedDateTime`。

可以简单地把`ZonedDateTime`理解成`LocalDateTime`加`ZoneId`。`ZoneId`是`java.time`引入的新的时区类。

要创建一个 `ZonedDateTime` 对象，有以下几种方法：

```java
/*方法一：通过 `now()` 方法返回当前时间*/
@Test
public void m2() {
  ZonedDateTime zbj = ZonedDateTime.now(); // 默认时区
  ZonedDateTime zny = ZonedDateTime.now(ZoneId.of("America/New_York")); //用指定时区获取当前时间
  System.out.println(zbj);    //2020-03-10T22:42:47.322+08:00[Asia/Shanghai]
  System.out.println(zny);    //2020-03-10T10:42:47.323-04:00[America/New_York]
}

/*方法二：通过给 `LocalDateTime` 附加一个 `ZoneId`*/
@Test
public void m2() {
  LocalDateTime ldt = LocalDateTime.of(2019, 9, 15, 15, 16, 17);
  ZonedDateTime zbj = ldt.atZone(ZoneId.systemDefault());
  ZonedDateTime zny = ldt.atZone(ZoneId.of("America/New_York"));
  System.out.println(zbj);	//2019-09-15T15:16:17+08:00[Asia/Shanghai]
  System.out.println(zny);	//2019-09-15T15:16:17-04:00[America/New_York]
}
```

> 方法一中：打印的两个 `ZonedDateTime` ，虽然时区不同，但表示的是同一时刻。
>
> 方法二中：以这种方式创建的`ZonedDateTime`，它的日期和时间与`LocalDateTime`相同，但附加的时区不同，因此是两个不同的时刻

### 3.1 时区转换

用`withZoneSameInstant()` 可以将关联时区转化到另一时区，转化后日期和时间都会相应调整。

```java
/*将北京时间调整为纽约时间*/
@Test
public void m1() {
  // 以中国时区获取当前时间:
  ZonedDateTime zbj = ZonedDateTime.now(ZoneId.of("Asia/Shanghai"));
  // 转换为纽约时间:
  ZonedDateTime zny = zbj.withZoneSameInstant(ZoneId.of("America/New_York"));
  System.out.println(zbj);    //2020-03-10T23:33:44.988+08:00[Asia/Shanghai]
  System.out.println(zny);    //2020-03-10T11:33:44.988-04:00[America/New_York]
}
```

> 涉及到时区时，千万不要自己计算时差，否则难以正确处理夏令时。

有了`ZonedDateTime` ，将其转化为本地时间就非常简单：

```java
ZonedDateTime zdt = ...
LocalDateTime ldt = zdt.toLocalDateTime();
```

## 4、Instant

计算机存储当前的时间，本质上只是一个不断增长的整数。

Java提供的`System.currentTimeMillis()`返回的就是以毫秒表示的当前时间戳。

在`java.time`中当前的时间戳以`Instant`类型表示，我们用`Instant.now()`获取当前时间戳：

```java
@Test
public void m1() {
  Instant now = Instant.now();
  System.out.println(now.getEpochSecond()); // 秒,1583850200
  System.out.println(now.toEpochMilli()); // 毫秒,1583850200151
}
```

在`Instant`内部只有两个核心字段：

```java
public final class Instant implements ... {
    private final long seconds;
    private final int nanos;
}
```

一个是以秒为单位的时间戳，另一个是更精确的纳秒精度。

给 `Instant` 附加上一个时区，就可以创建出 `ZonedDateTime` ：

```java
// 以指定时间戳创建Instant:
Instant ins = Instant.ofEpochSecond(1568568760);
ZonedDateTime zdt = ins.atZone(ZoneId.systemDefault());
System.out.println(zdt); // 2019-09-16T01:32:40+08:00[Asia/Shanghai]
```

对于某一个时间戳，给它关联上指定的`ZoneId`，就得到了`ZonedDateTime`，继而可以获得了对应时区的`LocalDateTime`。所以，`LocalDateTime`，`ZoneId`，`Instant`，`ZonedDateTime`和`long`都可以互相转换。

 ## 5、最佳实践

在使用日期和时间时，除非涉及到遗留代码，否则应该坚持使用新的API。

### 5.1 旧API ->新API

如果要把旧式的`Date`或`Calendar`转换为新API对象，可以通过`toInstant()`方法转换为`Instant`对象，再继续转换为`ZonedDateTime`：

```java
// Date -> Instant -> ZonedDateTime:
Date date = new Date();
Instant ins1 = date.toInstant();
ZonedDateTime zdt1 = ins1.atZone(ZoneId.systemDefault());

// Calendar -> Instant -> ZonedDateTime:
Calendar calendar = Calendar.getInstance();
Instant ins2 = calendar.toInstant();
ZonedDateTime zdt2 = ins2.atZone(calendar.getTimeZone().toZoneId());
```

> 从上面的代码还可以看到，旧的`TimeZone`提供了一个`toZoneId()`，可以把自己变成新的`ZoneId`。

### 5.2 新API ->旧API

如果要把新的`ZonedDateTime`转换为旧的API对象，只能借助`long`型时间戳做一个“中转”：

```java
// ZonedDateTime -> long:
ZonedDateTime zdt = ZonedDateTime.now();
long ts = zdt.toEpochSecond() * 1000;

// long -> Date:
Date date = new Date(ts);

// long -> Calendar:
Calendar calendar = Calendar.getInstance();
calendar.clear();
calendar.setTimeZone(TimeZone.getTimeZone(zdt.getZone().getId()));
calendar.setTimeInMillis(zdt.toEpochSecond() * 1000);
```

> 从上面的代码还可以看到，新的`ZoneId`转换为旧的`TimeZone`，需要借助`ZoneId.getId()`返回的`String`完成。

### 5.3 在数据库中存储日期和时间

除了旧式的`java.util.Date`，我们还可以找到另一个`java.sql.Date`，它继承自`java.util.Date`，但会自动忽略所有时间相关信息。

在数据库中，也存在几种日期和时间类型：

- `DATETIME`：表示日期和时间；
- `DATE`：仅表示日期；
- `TIME`：仅表示时间；
- `TIMESTAMP`：和`DATETIME`类似，但是数据库会在创建或者更新记录的时候同时修改`TIMESTAMP`。

在操作数据库时，我们需要把数据库类型与Java类型映射起来。下表是数据库类型与Java新旧API的映射关系：

| 数据库    | 对应Java类（旧）   | 对应Java类（新） |
| :-------- | :----------------- | :--------------- |
| DATETIME  | java.util.Date     | LocalDateTime    |
| DATE      | java.sql.Date      | LocalDate        |
| TIME      | java.sql.Time      | LocalTime        |
| TIMESTAMP | java.sql.Timestamp | LocalDateTime    |

实际上，在数据库中，我们需要存储的最常用的是时刻（`Instant`），因为有了时刻信息，就可以根据用户自己选择的时区，显示出正确的本地时间。所以，最好的方法是直接用长整数`long`表示，在数据库中存储为`BIGINT`类型。

下面是通过`timestampToString` 方法，将存储的`long` 型时间戳转化为不用用户的本地时间：

```java
public class Main {
    public static void main(String[] args) {
        long ts = 1583856181452L;
        System.out.println(timestampToString(ts, Locale.CHINA, "Asia/Shanghai"));
        System.out.println(timestampToString(ts, Locale.US, "America/New_York"));
    }

    static String timestampToString(long epochMilli, Locale lo, String zoneId) {
        Instant ins = Instant.ofEpochMilli(epochMilli);
        DateTimeFormatter f = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.MEDIUM, FormatStyle.SHORT);
        return f.withLocale(lo).format(ZonedDateTime.ofInstant(ins, ZoneId.of(zoneId)));
    }
}
```

# 九、并发

## 1、多线程基础

首先我们需要知道现代的操作系统，不管是windows、macOS、还是Linux，都是可以执行多任务的。否则你也不能边打开网页、边听着音乐，还登录着QQ。

在我们感觉上，cpu好像是在同时执行所有的任务，那是因为我们被计算机给“欺骗”了，在操作系统中，多个程序是被cpu交替执行的，也就是说，cpu会让浏览器执行0.001秒，让QQ执行0.001秒，再让音乐播放器执行0.001秒，只是这个时间太短，我们感觉不出来罢了。

即使对于今天的多核CPU，因为你的任务通常也是多于核数，所以任务仍然是交替执行。

### 1.1 进程

要学习线程，就必须得提到 `进程` 了，在计算机中，我们把一个任务称为一个进程，像你使用的浏览器就是一个进程。

然而在进程中又需要执行多个字任务。例如，我们在使用word时，可以一边打字，Word还会一边进行拼写检查，甚至还在后台打印，这种字任务就称为线程。

### 1.2 进程与线程

进程与线程的关系：

- 进程是分配和调度资源的独立单位，而线程是CPU调度的基本单元。

- `线程` 依存于 `进程` 而存在，没有单独存在的线程。所以进程结束后它拥有的所有线程也会被销毁，而线程的结束不会影响进程的存在。
- 一个进程可以有多个线程，但只会有一个线程。多个线程共享进程的资源。

### 1.3 工作模式

一个应用程序，既可以有多进程，可以多线程工作，因此实现多任务，就有以下几种方法：

- 多进程模式（每个进程只有一个线程）
- 多线程模式（一个进程有多个线程）
- 多进程 + 多线程模式（复杂度最高）

对于选择上面的哪种模式，就要考虑进程和线程的优缺点：

- 多进程的缺点：
  - 创建进程比创建线程开销更大；
  - 进程之间的通信比线程的通信要慢。
- 多进程的优点：
  - 多进程更加稳定，因为一个进程崩溃，不会影响到其他的进程。而在多线程的情况下，任何一个线程崩溃会导致整个进程崩溃。

## 2、创建线程

在 Java 中创建一个新线程是非常容易的，有以下方法：

- 方法一：从`Thread`派生一个自定义类，然后覆写`run()`方法：

  ```java
  /*方法一*/
  /*MyThread01.java*/
  public class MyThread01 extends Thread {
      @Override
      public void run() {
          System.out.println("启动了线程MyThread01");
      }
  }
  ```

  ```java
  @Test
  public void m0() {
    Thread myThread01 = new MyThread01();
    myThread01.start();	//启动新线程，`start()` 方法会自动调用覆写的 `run()` 方法
  }
  ```

- 创建一个 `Thread` 实例，传入一个 `Runnable` 实例：

  ```java
  /*MyThread02.java*/
  public class MyThread02 implements Runnable {
      @Override
      public void run() {
          System.out.println("启动了线程MyThread02");
      }
  }
  ```

  ```java
  @Test
  public void m1() {
    Thread myThread01 = new MyThread01();
    Thread myThread02= new Thread(new MyThread02());
    myThread01.start();
    myThread02.start();
  }
  ```

- 使用 `FutureTask` 配合 Thread

  ```java
  public class FutureTaskTest {
      public static void main(String[] args) throws ExecutionException, InterruptedException {
          FutureTask<Integer> task = new FutureTask<>(new Callable<Integer>() {
              @Override
              public Integer call() throws Exception {
                  System.out.println("running");
                  Thread.sleep(2000);
                  return 10;
              }
          });
  
          Thread task1 = new Thread(task, "task1");
          task1.start();
  
          System.out.println(task.get());
      }
  }
  ```

  ![image-20210208113833528](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210208113833528.png)

> Thread.java中的 `start()` 方法通知 “线程管理器” 此线程已经准备就绪，等待调用该线程的 `run()` 方法。具体什么时候调用，需要看系统的安排，有了异步执行的效果。因此不能直接调用 `thread.run()` 方法，否则就变成了同步，相当于调用了一个普通的方法。

### 2.1 Runnable接口

打开`Thread.java` ，我们可以发现，`Thread` 类实际上就是继承了 `Runnable` 接口，它们之间具有多态关系。

```java
public class Thread implements Runnable {
```

但是继承 `Thread` 类创建新线程的最大缺陷就是：Java不支持多继承，如果想要创建的线程类已经有了一个父类，这时就需要实现 `Runnable` 接口来对应。 

### 2.2 优先级

可以对线程设定优先级，设定优先级的方法是：

```java
Thread.setPriority(int n) // 1~10, 默认值5
```

优先级高的线程被操作系统调度的优先级较高，操作系统对高优先级线程可能调度更频繁，但我们决不能通过设置优先级来确保高优先级的线程一定会先执行。

## 3、线程的状态

在Java程序中，一个线程对象只能调用一次 `start()` 方法启动新线程，并在新线程中执行 `run()` 方法。一旦 `run()` 方法执行完毕，线程也就结束了。

![image-20210208170947996](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210208170947996.png)

Java线程的状态有以下几种：

- New：新创建的线程，还没有执行；
- Runnable：执行中的线程，正在执行 `run()` 方法中的代码； 

- Blocked：运行中的线程，因为某些操作被阻塞而挂起；
- Waiting：运行中的线程，因为某些操作在等待中，例如加锁等。
- Timed Waiting：运行中的线程，因为 `sleep()` 方法正在计时等待；
- Terminated：因为`run()`方法执行完毕，线程已终止。



1. 当线程启动之后，可以在`Runnable`、`Blocked`、`Waiting`和`Timed Waiting`这几个状态之间切换，直到最后变成 `Terminated` 状态，线程终止。
2. Java API 层面的Runnable 状态涵盖了操作系统层面的 【可运行状态】、【运行状态】和【阻塞状态】（由于BIO导致的线程阻塞，在Java中无法区分，仍热认为是可运行的）。

 

一个线程A 可以等待另外一个线程B直到B执行结束才开始执行。例如，`main` 线程等待 `myThread01` 线程结束才再运行： 

```java
@Test
public void m0() throws InterruptedException {
  Thread myThread01 = new Thread(new MyThread02());
  myThread01.start();
  myThread01.join();
  System.out.println("end");
}
```

> `join(long)` 的重载方法可以指定一个等待时间，超过等待时间之后就不再继续等待。

## 4、中断线程

中断线程有两种方法：

- 在其他线程中对目标线程调用 `interrupt()` 方法；
- 设置标志位，通过`running` 标志位判断线程是否应该继续执行。

方法一：调用 `interrupt()` 方法---==两阶段终止模式==

![image-20210208160348517](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210208160348517.png)

```java
public class InterruptedTest {
    public static void main(String[] args) throws InterruptedException {
        TwoPhaseTermination tpt = new TwoPhaseTermination();
        tpt.start();

        TimeUnit.SECONDS.sleep(6);
        tpt.stop();
    }
}

class TwoPhaseTermination {
    private Thread monitor;

    // 启动监控
    public void start() {
        monitor = new Thread(() -> {
            while (true) {
                Thread currentThread = Thread.currentThread();
                if (currentThread.interrupted()) {
                    System.out.println("善后工作");
                    break;
                }
                try {
                    TimeUnit.SECONDS.sleep(2);  //情况1
                    System.out.println("执行监控记录");   //情况2
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    currentThread.interrupt();  // 重新设置打断标志
                }
            }
        });
        monitor.start();
    }

    // 结束监控
    public void stop() {
        monitor.interrupt();
    }
}
```

> 在线程处于 sleep、wait和join时，如果调用 `Thread.interrupt`方法，`Thread.isInterrupted`方法会返回false；如果是正常运行的线程，`Thread.isInterrupted`方法会返回true。

方法二：通过设置`running` 标志位

```java
/*MyThread04.java*/
public class MyThread04 extends Thread {
	//使用volatile关键字标记
    public volatile boolean running = true;
    private int n = 0;

    @Override
    public void run() {
        while (running) {
            n++;
            System.out.println(n);
        }
    }
}
```

```java
/*测试*/
@Test
public void m1() throws InterruptedException {
  MyThread04 myThread04 = new MyThread04();
  myThread04.start();

  Thread.sleep(1);
  //中断线程
  myThread04.running = false;
  myThread04.join();
  System.out.println("================end==============");
}
```



## 5、守护线程

Java程序的入口就是`main` 线程，在`main` 线程中又可以启动其他线程。在所有的线程结束之后，JVM才会退出。

意思就是说：只要有一个线程没有退出，JVM进程就不会退出。

但是就有一种线程，它的目的就是无限循环，例如，一个执行定时任务的线程：

```java
class TimerThread extends Thread {
    @Override
    public void run() {
        while (true) {
            //执行任务
          	System.out.println(“定时任务执行”);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                break;
            }
        }
    }
}
```

根据代码可以知道上面这个线程只要启动了之后，就不会结束。那问题就是：谁负责结束这个线程呢？

事实上，不需要任何线程来负责它们的结束，我们可以使用守护线程（Daemon Thread）。

**守护线程**就是为其他线程服务的线程。在其他线程都执行结束后，虚拟机是不管守护线程是否结束的，会自动退出。

### 5.1 创建守护线程

方法和普通线程一样，只是在调用`start()`方法前，调用`setDaemon(true)`把该线程标记为守护线程：

```java
Thread t = new MyThread();
t.setDaemon(true);	//标记为守护线程
t.start();
```

> 需要注意的是：守护线程不能持有任何需要关闭的资源，例如打开文件等，因为虚拟机退出时，守护线程没有任何机会来关闭文件，这会导致数据丢失。

## 6、线程同步

在多线程同时运行时，线程的调度是由操作系统决定的，程序根本不能决定。就时候问题就来了：如果多个线程读、修改共享变量，就会出现数据不一致的问题。

下面的例子，一个线程相当于一个售票窗口，总共有5张票，每个窗口一次会卖出5张，正常来说，应该是有一个窗口会买不到票，但是如下程序结束后，2个窗口都卖出了票，余票变成了-5。

```java
public class Ticket {
    public static void main(String[] args) throws InterruptedException {
        SellThread t1 = new SellThread();
        t1.start();

        SellThread t2 = new SellThread();
        t2.start();

        t1.join();
        t2.join();
        System.out.println(Count.total);
    }
}

class Count {
    public static int total = 5;
}

class SellThread extends Thread {
    @Override
    public void run() {
        int total = Count.total;
        if (total > 0) {
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            Count.total = Count.total - 5;
            System.out.println("卖出5张票");
        }else {
            System.out.println("没有票了");
        }
    }
}
```

```java
//输出结果
卖出5张票
卖出5张票
余票为-5
```

上面的错误在于：

![](images/QQ20200312-130743.png)

要想解决对共享变量的读写问题，必须保证以原子的方法执行售票，即某一线程在执行时，其他线程必须等待：

![](images/QQ20200312-131643.png)

通过加锁和解锁的方式保证了一段代码的原子性，从而解决了共享变量的问题。在Java中，是使用 `synchronized` 关键字对一个对象进行加锁：

```java
synchronized(lock) {
    n = n + 1;
}
```

```java
/*用 synchronized 改写上面的代码*/
class SellThread extends Thread {
    @Override
    public void run() {
        synchronized (Count.lock) {	//获取锁
            int total = Count.total;
            if (total > 0) {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                Count.total = Count.total - 5;
                System.out.println("卖出5张票");
            }else {
                System.out.println("没有票了");
            }
        }	//释放锁
    }
}
```

> 两个线程在执行各自的 `synchronized(Counter.lock) { ... }` 代码块时，必须先获得锁，才能开始售票。
>
> 使用 `synchronized` 解决了多线程同步访问共享变量的问题。但是，它的缺点是带来了性能下降的问题。因此 `synchronized` 代码块不能并发执行，加锁和解锁也需要消耗时间。

在使用`synchronized`的时候，不必担心抛出异常。因为无论是否有异常，都会在`synchronized`结束处正确释放锁：

```java
public void add(int m) {
    synchronized (obj) {
        if (m < 0) {
            throw new RuntimeException();
        }
        this.value += m;
    } // 无论有无异常，都会在此释放锁
}
```

### 6.1 不需要synchronized的操作

JVM规范定义了几种原子操作：

- 基本类型（`long`和`double`除外）赋值，例如：`int n = m`；
- 引用类型赋值，例如：`List list = anotherList`。

例如：

```java
public void set(String s) {
    this.value = s;
}
```

但如果是多行赋值语句，就必须保证是同步操作。

## 7、死锁

对于多线程来说，必须要面临死锁的问题。看下面的场景：

- 线程1获得了资源A；
- 线程2获得了资源B。

随后（假设只有一个资源A、一个资源B，并且此时线程1并没有释放资源A，线程2也没有释放资源B），

- 线程1想要获得资源B，失败，等待。
- 线程2想要获得资源A，失败，等待。

这时候就发生了死锁，2个线程都在无限等待着。

对于死锁的解决，可以规定获取资源的顺序，即严格按照先获取资源A，再获取资源B的顺序。

## 8、使用wait和notify

`synchronized` 解决了多线程的竞争问题。多个线程同时向队列中添加任务，可以用 `synchronized` 加锁解决：

```java
class TaskQueue {
    Queue<String> queue = new LinkedList<>();

    public synchronized void addTask(String s) {
        this.queue.add(s);
    }
}
```

但是 `synchronized` 并没有解决多线程协调的问题。例如，生产线程和消费线程的协调问题。这时候就需要使用 `wait/notify` 机制。

```java
public class WaitNotifyTest {

    public static void main(String[] args) {
        TaskQueue queue = new TaskQueue(2);

        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                try {
                    String task = queue.getTask();
                    System.out.println("取出任务" + task);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, "Thread-"+i).start();
        }

        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                try {
                    queue.addTask("task-" + i);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}

class TaskQueue {
    private Queue<String> queue;
    private int size;
    private final static Object lock = new Object();

    public TaskQueue(int size) {
        queue = new LinkedList<>();
        this.size = size;
    }

    public void addTask(String str) throws InterruptedException {
        synchronized (lock) {
            while (queue.size() == size) {
                // 唤醒在等待的线程
                lock.wait();
            }
            queue.add(str);
            System.out.println("添加任务" + str);
            lock.notifyAll();
        }
    }

    public String getTask() throws InterruptedException {
        synchronized (lock) {
            // 当条件不满足时，线程就等待
            while (queue.isEmpty()) {
                // 释放锁
                lock.wait();
                // 重新获取锁
            }
            String remove = queue.remove();
            lock.notifyAll();
            return remove;
        }
    }
}
```

> 对于 `wait()` 需要注意的是：
>
> - 该方法必须在当前获取的锁对象上调用，上面获取的锁是 `this` 锁，因此调用 `this.wait()` 。
> - 在 `wait()` 方法调用时，放释放线程获得的锁，`wait()` 方法返回后，会重新试图获取锁。
>
> 对于 `notifyAll()` 和 `notify()` 需要注意的是：
>
> - 使用`notifyAll()`将唤醒所有当前正在`this`锁等待的线程，而`notify()`只会唤醒其中一个。通常使用 `notifyAll()` 更加安全，因为我们我们考虑不周，会有些线程永远没有被唤醒。
>
> - 假设现在有3个线程被唤醒，但是也只有一个能获取到 `this` 锁，剩下两个将继续等待。再次被唤醒时，需要再判断队列是否为空，所以下面的写法是错误的：
>
>   ```java
>   /*错误写法*/
>       public String getTask() throws InterruptedException {
>           synchronized (lock) {
>               if (queue.isEmpty()) {
>                   lock.wait();
>               }
>               String remove = queue.remove();
>               lock.notifyAll();
>               return remove;
>           }
>       }
>   ```
>

>  `wait()` `notify()` `notifyAll()` 方法都属于 `Object` 类的一部分，不属于 `Thread`

> **wait() 和 sleep() 的区别**
>
> - wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
> - wait() 会释放锁，sleep() 不会。

## 9、使用park和unpark

- `LockSupport.park();` 	暂停当前线程
- `LockSupport.unpark(thread);`  恢复某个线程的运行

```java
/** 先 park 后 unpark */
public class ParkUnParkTest {

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            System.out.println("start...");

            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println("park...");
            LockSupport.park();
            System.out.println("resume...");
        });
        thread.start();

        TimeUnit.SECONDS.sleep(5);
        System.out.println("unpark");
        LockSupport.unpark(thread);
    }
}

/** 
start...
park...
unpark
resume...
*/
```

```java
/** 先 unpark 后 park */
public class ParkUnParkTest {

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            System.out.println("start...");

            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println("park...");
            LockSupport.park();
            System.out.println("resume...");
        });
        thread.start();

        TimeUnit.SECONDS.sleep(1);
        System.out.println("unpark");
        LockSupport.unpark(thread);
    }
}

/** 
start...
unpark
park...
resume...
*/
```

> - wait，notify 和 notifyAll 必须配合 Object Monitor 一起使用，而 park，unpark 不必
> - park & unpark 是以线程为单位来【阻塞】和【唤醒】线程，而 notify 只能随机唤醒一个等待线程，notifyAll 是唤醒所有等待线程，就不那么【精确】
> - park & unpark 可以先 unpark，而 wait & notify 不能先 notify



### 原理

每个线程都有自己的一个parker对象，由3部分组成：`_counter` `_cond` `_mutex` 

#### 先park后unpark

![image-20210209114715823](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210209114715823.png)

1. 当前线程调用 `unsage.park` 方法
2. 检查 `_counter` ，在本情况中为0，这时获得 `_mutex` 互斥锁
3. 线程进入 `_cond` 条件变量 阻塞
4. 设置 `_counter = 0`

![image-20210209115005255](images/image-20210209115005255.png) 

1. 调用 `Unsafe.unpark(Thread_0)` 方法，设置 `_counter = 1 `
2. 唤醒 `_cond` 条件变量中的 `Thread_0 `
3. `Thread_0` 恢复运行
4. 设置 `_counter = 0`

#### 先unpark后park

![image-20210209115230351](images/image-20210209115230351.png)

1. 调用 `Unsafe.unpark(Thread_0)` 方法，设置 `_counter = 1` 
2. 当前线程调用 `Unsafe.park()` 方法
3. 检查 `_counter` ，在本情况中为 1，这时线程无需阻塞，继续运行
4. 设置 `_counter = 0` 

## 10、AQS

### 10.1 概念

AQS全称是 `AbstractQueuedSynchronizer`，是阻塞式锁和相关的同步器工具的框架。

特点：

- 用 state 属性来表示资源的状态（分为独占模式和共享模式），子类需要定义如何维护这个状态，控制如何获取锁和设置锁。
  - `getState` ：获取 state 状态
  - `setState` ：设置 state 状态
  - `compareAndSetState` ：CAS机制设置 state 状态
  - 独占模式 是只有一个线程能够访问资源，共享模式 可以允许多个线程访问资源
- 提供了基于 FIFO 的等待队列，类似于 Monitor 的 EntryList
- 条件变量来实现等待、唤醒机制，支持多个条件变量，类似于 Monitor 的 WaitSet

子类主要实现这样一些方法（默认抛出 `UnsupportedOperationException`） 

- `tryAcquire`
- `tryRelease`
- `tryAcquireShared` 
- `tryReleaseShared` 
- `isHeldExclusively`

获取锁的方法：

```java
// 如果获取锁失败
if (!tryAcquire(arg)) {
 // 入队, 可以选择阻塞当前线程 park unpark
}
```

释放锁的方法：

```java
// 如果释放锁成功
if (tryRelease(arg)) {
 // 让阻塞线程恢复运行
}
```

### 10.2 自定义实现不可重入锁

```java
// 自定义锁（不可重入锁），实现参考 AbstractQueuedSynchronizer 的注释
class MyLock implements Lock {

    private static class Sync extends AbstractQueuedSynchronizer {
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        // Acquires the lock if state is zero
        public boolean tryAcquire(int acquires) {
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        // Releases the lock by setting state to zero
        protected boolean tryRelease(int releases) {
            if (getState() == 0) throw new IllegalMonitorStateException();
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        // Provides a Condition
        Condition newCondition() { return new ConditionObject(); }
    }

    private final Sync sync = new Sync();

    // 尝试加锁，不成功，则进入等待队列
    @Override
    public void lock() {
        sync.acquire(1);
    }

    // 尝试，不成功，进入等待队列，可打断
    @Override
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    @Override
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(time));
    }

    @Override
    public void unlock() {
        sync.release(1);
    }

    // 生成条件变量
    @Override
    public Condition newCondition() {
        return sync.newCondition();
    }
}
```

测试一下：

```java
public class Test2 {
    public static void main(String[] args) {
        MyLock lock = new MyLock();
        new Thread(() -> {
            lock.lock();
            try {
                System.out.println("t1---locking...");
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                System.out.println("t1---unlocking");
                lock.unlock();
            }
        }, "t1").start();

        new Thread(() -> {
            lock.lock();
            try {
                System.out.println("t2---locking...");
            } finally {
                System.out.println("t2---unlocking");
                lock.unlock();
            }
        }, "t2").start();
    }
}

/**
t1---locking...
t1---unlocking
t2---locking...
t2---unlocking
*/
```

### 10.3 主要用到AQS的并发工具类

![image-20210209215244126](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210209215244126.png)

## 11、使用ReentrantLock

从Java 5开始，引入了一个高级的处理并发的`java.util.concurrent`包，它提供了大量更高级的并发功能，能大大简化多线程程序的编写。

虽然Java语言直接提供的 `synchronized` 关键字能用于加锁，但是这种锁一是重，而是获取的时候必须一直等待，没有额外的尝试机制。

`java.util.concurrent.locks`包提供的`ReentrantLock`用于替代`synchronized`加锁。

```java
public class Counter {
    private final Lock lock = new ReentrantLock();
    private int count;

    public void add(int n) {
        lock.lock();
        try {
            count += n;
        } finally {
            lock.unlock();
        }
    }
}
```

> `ReentrantLock`是可重入锁【一条线程能够反复进入被它自己持有锁的同步块中】，它和`synchronized`一样，一个线程可以多次获取同一个锁。

### 11.1 和`synchronized`不同

和`synchronized`不同的是，`ReentrantLock` 可以尝试获取锁：

```java
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try {
        ...
    } finally {
        lock.unlock();
    }
}
```

上述代码在尝试获取锁的时候，最多等待1秒。如果1秒后仍未获取到锁，`tryLock()`返回`false`，程序就可以做一些额外处理，而不是无限等待下去。

所以，使用`ReentrantLock`比直接使用`synchronized`更安全，线程在`tryLock()`失败的时候不会导致死锁。

ReentrantLock与synchronized相比增加了一些高级功能，主要有以下三项：

- `等待可中断`：是指当持有锁的线程长期不释放锁的时候，正在等待的线程**可以选择放弃等待**，改为处理其他事情。可中断特性对处理执行时间非常长的同步块很有帮助。
- `公平锁`：是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁；而非公平锁则不保证这一点，在锁被释放时，任何一个等待锁的线程都有机会获得锁。`synchronized`中的锁是非公平的，`ReentrantLock`在默认情况下也是非公平的，但可以通过带布尔值的构造函数要求使用公平锁。不过一旦使用了公平锁，将会导致`ReentrantLock`的性能急剧下降，会明显影响吞吐量。
- 锁绑定多个条件：是指一个`ReentrantLock`对象可以同时绑定多个`Condition`对象。在`synchronized`中，锁对象的`wait()`跟它的`notify()`或者`notifyAll()`方法配合可以实现一个隐含的条件，如果要和多于一个的条件关联的时候，就不得不额外添加一个锁；而`ReentrantLock`则无须这样做，多次调用`newCondition()`方法即可。

### 11.2 和`synchronized`选择

> 来自《深入理解Java虚拟机》

推荐在`synchronized`与`ReentrantLock`都可满足需要时优先使用`synchronized`：

- `synchronized`是在Java语法层面的同步，足够清晰，也足够简单。每个Java程序员都熟悉`synchronized`，但J.U.C中的Lock接口则并非如此。因此在只需要基础的同步功能时，更推荐`synchronized`。
- Lock应该确保在finally块中释放锁，否则一旦受同步保护的代码块中抛出异常，则有可能永远不会释放持有的锁。这一点必须由程序员自己来保证，而使用synchronized的话则==可以由Java虚拟机来确保即使出现异常，锁也能被自动释放==。
- 尽管在JDK 5时代`ReentrantLock`曾经在性能上领先过`synchronized`，但这已经是十多年之前的胜利了。从长远来看，Java虚拟机更容易针对`synchronized`来进行优化，因为Java虚拟机可以在线程和对象的元数据中记录`synchronized`中锁的相关信息，而使用J.U.C中的Lock的话，Java虚拟机是很难得知具体哪些锁对象是由特定线程锁持有的。

### 11.3 使用Condition

从上节知道，使用`ReentrantLock`比直接使用`synchronized`更安全，可以替代`synchronized`进行线程同步。但是在处理多线程协调问题时，我们就需要使用 `Conditon` 来实现 `wait` 和 `notify` 的功能。

下面通过 `ReentrantLock` 和 `Condition` 来实现 `TaskQueue`：

```java
public class TaskQueue {
    private final Lock lock = new ReentrantLock();
    private final Condition condition = lock.newCondition();
    private Queue<String> queue = new LinkedList<>();

    public void addTask(String s) {
        lock.lock();
        try {
            queue.add(s);
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public String getTask() {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                condition.await();
            }
            return queue.remove();
        } finally {
            lock.unlock();
        }
    }
}
```

> 使用`Condition`时，引用的`Condition`对象必须从`Lock`实例的`newCondition()`返回，这样才能获得一个绑定了`Lock`实例的`Condition`实例。
>
> 类似的，`Condition`提供了`await()`、`signal()`、`signalAll()`。
>
> `await()`可以在等待指定时间后，如果还没有被唤醒，可以自己醒来：
>
> ```
> if (condition.await(1, TimeUnit.SECOND)) {
>  // 被其他线程唤醒
> } else {
>  // 指定时间内没有被其他线程唤醒
> }
> ```
>
> 通过使用`Condition`配合`Lock`，我们可以实现更灵活的线程同步。

### 11.4 ReentrantLock原理

![image-20210209220153907](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210209220153907.png)

#### 非公平锁原理

##### 加锁解锁流程

```java
/** ReentrantLock构造方法，默认为非公平锁 */
public ReentrantLock() {
    sync = new NonfairSync();
}
```

在没有竞争时

![image-20210209220614942](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210209220614942.png)

第一个竞争出现时

![image-20210209220658253](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210209220658253.png)

`Thread-1` 执行了：

1.  CAS 尝试将 state 由 0 改为 1，结果失败
2. 进入 tryAcquire 逻辑，这时 state 已经是1，结果仍然失败
3. 接下来进入 addWaiter 逻辑，构造 Node 队列
   1. 图中黄色三角表示该 Node 的 waitStatus 状态，其中 0 为默认正常状态
   2. Node 的创建是懒惰的
   3. 其中第一个 Node 称为 Dummy（哑元）或哨兵，用来占位，并不关联线程

![image-20210209221308052](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210209221308052.png)

 

#### 公平锁原理



## 12、使用ReadWriteLock

前面的那些，不管是 `synchronized`，还是 `ReentrantLock`，都保证了只有一个线程能够执行临界区代码。但有时候这种保护有点过头。例如，在`add()` 方法中因为要修改变量，所以应该加锁，但是在 `get()` 方法中只读取数据，不会修改数据，此时允许多个线程同时调用不会有问题。

我们想要实现的状态如下：

|      | 读     | 写     |
| :--- | :----- | :----- |
| 读   | 允许   | 不允许 |
| 写   | 不允许 | 不允许 |

使用 `ReadWriteLock` 可以实现该状态，保证了：

- 只允许一个线程写入——其他线程既不允许读，也不允许写
- 允许多个线程同时读（提高性能）

```java
public class Counter {
    private final ReadWriteLock rwlock = new ReentrantReadWriteLock();
    private final Lock rlock = rwlock.readLock();	//获取读锁
    private final Lock wlock = rwlock.writeLock();	//获取写锁
    private int[] counts = new int[10];

    public void inc(int index) {
        wlock.lock(); // 加写锁
        try {
            counts[index] += 1;
        } finally {
            wlock.unlock(); // 释放写锁
        }
    }

    public int[] get() {
        rlock.lock(); // 加读锁
        try {
            return Arrays.copyOf(counts, counts.length);
        } finally {
            rlock.unlock(); // 释放读锁
        }
    }
}
```

> 允许多个线程同时获得读锁，提高了并发读得效率。



## 13、使用StampedLock

在前面的 `ReadWriteLock` 中，可以发现：如果有线程在读，写线程需要等待读线程释放锁之后才能获取写锁，因为读的过程中不允许写，这是一种**悲观的读锁**。

为了进一步提高并发执行效率，Java8引入了新的写锁：`StampedLock`。

`StampedLock` ：在读的过程中允许获取写锁写入，但是这样以来，读取到的数据就有可能不一致，所以，还需要一点额外的代码来判断读的过程中是否有写入，这种读锁是一种**乐观锁**。

- 乐观锁：估计读的过程中不会写，十分的乐观。
- 悲观锁：认为读的过程中总会发生写。

```java
public class Point {
    private final StampedLock stampedLock = new StampedLock();

    private double x;
    private double y;

    public void move(double deltaX, double deltaY) {
        long stamp = stampedLock.writeLock(); // 获取写锁
        try {
            x += deltaX;
            y += deltaY;
        } finally {
            stampedLock.unlockWrite(stamp); // 释放写锁
        }
    }

    public double distance() throws InterruptedException {
        long stamp = stampedLock.tryOptimisticRead();
        // 注意下面两行代码不是原子操作
        // 假设x,y = (100,200)
        double currentX = x;
        // 此处已读取到x=100，但x,y可能被写线程修改为(300,400)
        double currentY = y;
        // 此处已读取到y，如果没有写入，读取是正确的(100,200)
        // 如果有写入，读取是错误的(100,400)
        Thread.sleep(5);
        if (!stampedLock.validate(stamp)) { // 检查乐观读锁后是否有其他写锁发生
            System.out.println("有线程写入");
            stamp = stampedLock.readLock(); // 获取一个悲观读锁
            try {
                currentX = x;
                currentY = y;
            } finally {
                stampedLock.unlockRead(stamp); // 释放悲观读锁
            }
        }
        return Math.sqrt(currentX * currentX + currentY * currentY);

    }

}
```

> 在读取时，我们通过`tryOptimisticRead()`获取一个乐观读锁，并返回版本号。接着进行读取，读取完成后，我们通过`validate()`去验证版本号，如果不成功，说明有数据写入，需要重新读取。由于写入的概率不高，在大多数情况下通过乐观锁就可以获取数据，极少使用悲观锁。
>
> `StampedLock`把读锁细分为乐观读和悲观读，优点是能进一步提升并发效率。缺点是：
>
> - 代码更加复杂；
> - `StampedLock`是不可重入锁，不能在一个线程中反复获取同一个锁。



## 14、线程安全集合类

线程安全的集合类主要分为3种：

![image-20210223091723582](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210223091723582.png)

> Blocking类：大部分的实现时基于锁，并提供用来阻塞的方法；
>
> CopyOnWrite类：容器修改时开销会较重；
>
> Concurrent类：
>
> - 内部很多操作使用CAS实现，一般具有很高的吞吐量；
> - 弱一致性
>   - 遍历时弱一致性，例如，在使用迭代器遍历时，如果容器发生修改，迭代器仍然可以运行，但数据时旧的；
>   - 求大小弱一致性，size操作未必正确；
>   - 读取弱一致性；

针对`List`、`Map`、`Set`、`Deque`等，`java.util.concurrent`包也提供了对应的并发集合类。

| interface | non-thread-safe         | 线程安全                                 |
| :-------- | :---------------------- | :--------------------------------------- |
| List      | ArrayList               | CopyOnWriteArrayList                     |
| Map       | HashMap                 | ConcurrentHashMap                        |
| Set       | HashSet / TreeSet       | CopyOnWriteArraySet                      |
| Queue     | ArrayDeque / LinkedList | ArrayBlockingQueue / LinkedBlockingQueue |
| Deque     | ArrayDeque / LinkedList | LinkedBlockingDeque                      |

`java.util.Collections`工具类还提供了一个旧的线程安全集合转换器，可以这么用：

```
Map unsafeMap = new HashMap();
Map threadSafeMap = Collections.synchronizedMap(unsafeMap);
```

但是它实际上是用一个包装类包装了非线程安全的`Map`，然后对所有读写方法都用`synchronized`加锁，这样获得的线程安全集合的性能比`java.util.concurrent`集合要低很多，所以不推荐使用。

![image-20210223091533155](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210223091533155.png)

### ConcurrentHashMap

#### JDK1.7

在JDK1.7中，ConcurrentHashMap 是由 Segment 数组和 HashEntry 数组构成，采用分段锁来保证安全性。

Segment 是 ReentrantLock 重入锁，在 ConcurrentHashMap 中扮演锁的角色；HashEntry 则用于存储键值对数据。

一个 ConcurrentHashMap 里包含一个 Segment 数组，一个 Segment 里包含一个 HashEntry 数组，Segment 的结构和 HashMap 类似，是一个数组和链表结构。

![image-20210223105732093](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210223105732093.png)

#### JDK1.8

在JKD1.8中，已经摒弃了 Segment 的概念，而是直接用 Node 数组+链表+红黑树的数据结构来实现，并发控制使用 Synchronized 和 CAS 来操作，整个看起来就像是优化过且线程安全的 HashMap，虽然在 JDK1.8 中还能看到 Segment 的数据结构，但是已经简化了属性，只是为了兼容旧版本。

![image-20210223105940234](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210223105940234.png)

```java
/*控制标识符，用来控制table的初始化和扩容的操作，不同的值有不同的含义
    *当为负数时：-1代表正在初始化，-N代表有N-1个线程正在 进行扩容
    *当为0时：代表当时的table还没有被初始化
    *当为正数时：表示初始化或者下一次进行扩容的大小
    */
private transient volatile int sizeCtl;

// 整个 ConcurrentHashMap 就是一个 Node[]
static class Node<K,V> implements Map.Entry<K,V> {}

// hash 表
transient volatile Node<K,V>[] table;

// 扩容时的 新 hash 表
private transient volatile Node<K,V>[] nextTable;

// 扩容时如果某个 bin 迁移完毕, 用 ForwardingNode 作为旧 table bin 的头结点
static final class ForwardingNode<K,V> extends Node<K,V> {}

// 用在 compute 以及 computeIfAbsent 时, 用来占位, 计算完成后替换为普通 Node
static final class ReservationNode<K,V> extends Node<K,V> {}

// 作为 treebin 的头节点, 存储 root 和 first
static final class TreeBin<K,V> extends Node<K,V> {}

// 作为 treebin 的节点, 存储 parent, left, right
static final class TreeNode<K,V> extends Node<K,V> {}
```

#### JDK1.8中为什么要摒弃分段锁

- **jdk1.8中锁的粒度更细了**。jdk1.7中ConcurrentHashMap 的concurrentLevel（并发数）基本上是固定的。jdk1.8中的concurrentLevel是和数组大小保持一致的，每次扩容，并发度扩大一倍.

- **红黑树的引入**，对链表的优化使得 hash 冲突时的 put 和 get 效率更高

- **获得JVM的支持** ，ReentrantLock 毕竟是 API 这个级别的，后续的性能优化空间很小。 synchronized 则是 JVM 直接支持的， JVM 能够在运行时作出相应的优化措施：锁粗化、锁消除、锁自旋等等。这就使得 synchronized 能够随着 JDK 版本的升级而不改动代码的前提下获得性能上的提升。

参考：[ConcurrentHashMap 原理浅析](https://juejin.cn/post/6844904018729254926#heading-0)

### BlockingQueue

`BlockingQueue` 即阻塞队列，被阻塞的情况主要有如下两种：

- 当队列满了的时候执行入队操作；
- 当队列空了的时候执行出队操作；

在Java中，`BlockingQueue`的接口位于`java.util.concurrent` 包中(在Java5版本开始提供)，由上面介绍的阻塞队列的特性可知，阻塞队列是线程安全的。

阻塞队列主要是用在生产者/消费者的场景

![image-20210207102725488](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210207102725488.png)

负责生产的线程不断的制造新对象并插入到阻塞队列中，直到达到这个队列的上限值。队列达到上限值之后生产线程将会被阻塞，直到消费的线程对这个队列进行消费。同理，负责消费的线程不断的从队列中消费对象，直到这个队列为空，当队列为空时，消费线程将会被阻塞，除非队列中有新的对象被插入。

#### BlockingQueue接口中的方法

- 插入方法：
  - add(E e) : 添加成功返回true，失败抛IllegalStateException异常
  - offer(E e) : 成功返回 true，如果此队列已满，则返回 false。
  - offer(E e, long timeout, TimeUnit unit) : 如果操作不能马上进行，操作会被阻塞指定的时间。
  - put(E e) :将元素插入此队列的尾部，如果该队列已满，则一直阻塞
- 删除方法:
  - remove(Object o) :移除指定元素,成功返回true，失败返回false
  - poll() : 获取并移除此队列的头元素，若队列为空，则返回 null
  - poll(long timeout, TimeUnit unit) : 获取并移除此队列的头元素，若没有元素则等待指定时间。
  - take()：获取并移除此队列头元素，若没有元素则一直阻塞。

- 检查方法
  - element() ：获取但不移除此队列的头元素，没有元素则抛异常
  - peek() :获取但不移除此队列的头；若队列为空，则返回 null。

> 需要注意的是，我们不能向BlockingQueue中插入`null`，否则会报`NullPointerException`。

> 阻塞队列一般是基于ReentrantLock的Condition机制来实现

#### ArrayBlockingQueue

`ArrayBlockingQueue`是一个有边界的阻塞队列，它的内部实现是一个数组。有边界的意思是它的容量是有限的，我们必须在其初始化的时候指定它的容量大小，容量大小一旦指定就不可改变。

`ArrayBlockingQueue`是以先进先出的方式存储数据，最新插入的对象是尾部，最新移出的对象是头部。下面是一个初始化和使用`ArrayBlockingQueue`的例子：

```java
BlockingQueue queue = new ArrayBlockingQueue(1024);
queue.put("1");
Object object = queue.take();
```

下面是一个自实现的`ArrayBlockingQueue`，从这个例子中可以看出 `ArrayBlockingQueue` 的实现原理

```java
public class CustomArrayBlockingQueue {
    private final Object[] items = new Object[100];
    private final Lock lock = new ReentrantLock();
    private final Condition notEmpty = lock.newCondition();
    private final Condition notFull = lock.newCondition();

    private int putIndex = 0;
    private int takeIndex = 0;
    private int count = 0;

    public void put(Object item) throws InterruptedException {
        if (item == null) {
            throw new NullPointerException();
        }
        lock.lock();
        try {
            while (count == items.length)
                notFull.await();
            items[putIndex] = item;
            if (++putIndex == items.length)
                putIndex = 0;
            count++;
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    public Object take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0)
                notEmpty.await();
            Object item = items[takeIndex];
            items[takeIndex] = null;
            if (++takeIndex == items.length)
                takeIndex = 0;
            count--;
            notFull.signal();
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```



#### LinkedBlockingQueue

`LinkedBlockingQueue`阻塞队列大小的配置是可选的，如果我们初始化时指定一个大小，它就是有边界的，如果不指定，它就是无边界的。说是无边界，其实是采用了默认大小为`Integer.MAX_VALUE`的容量 。它的内部实现是一个链表。

和`ArrayBlockingQueue`一样，`LinkedBlockingQueue` 也是以先进先出的方式存储数据，最新插入的对象是尾部，最新移出的对象是头部。下面是一个初始化和使用`LinkedBlockingQueue`的例子：

```java
BlockingQueue<String> unbounded = new LinkedBlockingQueue<String>();
BlockingQueue<String> bounded   = new LinkedBlockingQueue<String>(1024);
bounded.put("Value");
String value = bounded.take();
```

`LinkedBlockingQueue` 的实现要点：

![image-20210207133004612](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210207133004612.png)

#### PriorityBlockingQueue

`PriorityBlockingQueue`是一个没有边界的队列，它的排序规则和 `java.util.PriorityQueue`一样。需要注意，`PriorityBlockingQueue`中允许插入null对象。

所有插入`PriorityBlockingQueue`的对象必须实现 `java.lang.Comparable`接口，队列优先级的排序规则就是按照我们对这个接口的实现来定义的。

> 另外，我们可以从PriorityBlockingQueue获得一个迭代器Iterator，但这个迭代器并不保证按照优先级顺序进行迭代。

#### DelayQueue

**DelayQueue**阻塞的是其内部元素，DelayQueue中的元素必须实现 `java.util.concurrent.Delayed`接口，这个接口的定义非常简单：

```
public interface Delayed extends Comparable<Delayed> {
long getDelay(TimeUnit unit);
}
```

`getDelay()`方法的返回值就是队列元素被释放前的保持时间，如果返回`0`或者一个`负值`，就意味着该元素已经到期需要被释放，此时DelayedQueue会通过其`take()`方法释放此对象。

从上面Delayed 接口定义可以看到，它还继承了`Comparable`接口，这是因为DelayedQueue中的元素需要进行排序，一般情况，我们都是按元素过期时间的优先级进行排序。

```java
public class DelayedElement implements Delayed {

    private String name;
    private long delay;
    private long expired;

    public DelayedElement(String name, long delay) {
        this.name = name;
        this.delay = delay;
        this.expired = delay + System.currentTimeMillis();
    }

    @Override
    public long getDelay(TimeUnit unit) {
        return (expired - System.currentTimeMillis());
    }

    @Override
    public int compareTo(Delayed o) {
        DelayedElement ele = (DelayedElement) o;
        return getExpired() > ele.getExpired() ? 1 : -1;
    }

    public long getExpired() {
        return expired;
    }

    @Override
    public String toString() {
        return "DelayedElement{" +
                "name='" + name + '\'' +
                ", delay=" + delay +
                ", expired=" + expired +
                '}';
    }

    public static void main(String[] args) throws InterruptedException {
        DelayQueue<DelayedElement> queue = new DelayQueue<>();
        DelayedElement ele1 = new DelayedElement("cache 5 seconds", 5000);
        DelayedElement ele2 = new DelayedElement("cache 3 seconds", 3000);
        DelayedElement ele3 = new DelayedElement("cache 2 seconds", 2000);
        DelayedElement ele4 = new DelayedElement("cache 4 seconds", 4000);
        queue.put(ele1);
        queue.put(ele2);
        queue.put(ele3);
        queue.put(ele4);
        while (!queue.isEmpty()) {
            System.out.println(queue.take());
        }
    }
}

/*
DelayedElement{name='cache 2 seconds', delay=2000, expired=1612673359131}
DelayedElement{name='cache 3 seconds', delay=3000, expired=1612673360131}
DelayedElement{name='cache 4 seconds', delay=4000, expired=1612673361131}
DelayedElement{name='cache 5 seconds', delay=5000, expired=1612673362131}
*/
```

#### SynchronousQueue

`SynchronousQueue`是一个没有数据缓冲的`BlockingQueue`，生产者线程对其的插入操作put必须等待消费者的移除操作take，反过来也一样。

> **SynchronousQueue默认是不公平的，其不公平性体现在哪里？**
>
> 比如线程1向队列中存入了一个元素，之后被阻塞住；然后线程2向队列中存入了一个元素，也被阻塞住了；之后线程3来取元素，先取出的会是线程2中的元素，这里就体现了不公平性，线程2中的数据后被存入的，却先被取出来。
>
> 再比如线程1先从队列中取出一个元素，之后线程2也从队列中取出一个元素；之后线程3向队列中存元素，这时线程2会拿到线程3存入的元素，而不是线程1，这也是不公平的体现。

[并发编程之 SynchronousQueue 核心源码分析](https://juejin.cn/post/6844903600410329102)

## 15、使用Atomic

Java的 `java.util.concurrent` 包除了提供底层锁、并发集合，还提供了一组原子操作的封装类，它们位于 `java.util.concurrent.atomic` 包。

以 `AtomicInteger` 为例，它的主要操作有：

- 增加值并返回新值：`int addAndGet(int delta)`
- 加1后返回新值：`int incrementAndGet()`
- 获取当前值：`int get()`
- 用CAS方式设置：`int compareAndSet(int expect, int update)`

Atomic类是通过无锁（lock-free）的方式实现的线程安全（thread-safe）访问。它的主要原理是利用了CAS：Compare and Set。

如果我们自己通过CAS编写`incrementAndGet()`，它大概长这样：

```java
public int incrementAndGet(AtomicInteger var) {
  int pre, next;
  do {
    prev = var.get();
    next = prev + 1;
  } while(!var.compareAndSet(prev, next));
  return next;
}
```

CAS是指，在 `var.compareAndSet(prev, next)` 操作中，如果`var`的当前值是`prev`，那么就更新为`next`，返回`true`。如果不是`prev` ，说明值发生了修改，就什么也不干，返回 `false`，然后重新获取值，修改。

通过CAS操作并配合 `do...while` 循环，即使其他线程修改了 `AtomicInteger` 的值，最终的结果也正确。



通常情况下，我们不需要自己实现复杂的并发操作，而是用 `incrementAndGet()` 这样封装好的方法。

在高度竞争的情况下，还可以使用Java8提供的`LongAdder`和`LongAccumulator`。

## 16、使用线程池

使用线程池的原因是：创建线程需要操作系统资源（线程资源、栈空间等），频繁创建和销毁大量线程需要消耗大量时间。

我们希望能够把很多的小任务让一组线程执行，而不是一个任务用一个线程执行。这一组线程就是线程池。

在线程池内维护了若干线程，没有任务时，这些线程就出了等待状态。如果有新任务，就分配一个空闲线程执行。如果这时所有线程都在忙碌，有两种处理方法：

- 让新任务等待；
- 在创建一个新线程处理。

在Java标准库中提供了 `ExecutorService` 接口表示线程池，它的常用实现类有：

- `FixedThreadPool`：线程数固定的线程池；
- `CachedThreadPool`：线程数根据任务动态调整的线程池；
- `SingleThreadExecutor`：仅单线程执行的线程池。

线程池的典型用法如下：

```java
// 创建固定大小的线程池:
ExecutorService executor = Executors.newFixedThreadPool(3);
// 提交任务:
executor.submit(task1);
executor.submit(task2);
executor.submit(task3);
executor.submit(task4);
executor.submit(task5);
//关闭线程池
executor.shutdown();
```

> 线程池在程序结束的时候要关闭。关闭的方法有3种：
>
> - `shutdown()`：等正在执行的任务完成后，再关闭线程池。
> - `shutdownNow()` ：立刻停止正在执行的任务。
> - `awaitTermination()` ：等待指定的时间关闭线程池。

下面我们以`FixedThreadPool`为例，看看线程池的执行逻辑：

```java
public class Main {
    public static void main(String[] args) {
        // 创建一个固定大小为4的线程池:
        ExecutorService es = Executors.newFixedThreadPool(4);
        for (int i = 0; i < 8; i++) {
          	//一次性放入8个任务，先会执行4个，在线程空闲后执行后面4个
            es.submit(new Task("" + i));
        }
        // 关闭线程池:
        es.shutdown();
    }
}

class Task extends Thread {

    private String name;

    public Task(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        System.out.println("启动任务：" + name);

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("执行结束：" + name);
    }
}
```

如果我们把线程池控制在 4~10个之间动态调整该怎么办？打开 `Executors.newCachedThreadPool()`方法的源码：

```java
public static ExecutorService newCachedThreadPool() {
  return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                60L, TimeUnit.SECONDS,
                                new SynchronousQueue<Runnable>());
}
```

因此，我们可以这样写：

```java
ExecutorService es = new ThreadPoolExecutor(4, 10,
                                60L, TimeUnit.SECONDS,
                                new SynchronousQueue<Runnable>());
```

### 16.1 ScheduledThreadPool

`ScheduledThreadPool` 是针对需要定期的、反复执行的任务。例如，每5分钟报时。

创建一个 `ScheduledThreadPool `仍然是通过`Executors`类：

```java
ScheduledExecutorService ses = Executors.newScheduledThreadPool(4);
```

虽然是定期重复执行，但也有这3种执行状态：

- 特定延迟后只执行一次：

  ```java
  // 1秒后执行一次性任务:
  ses.schedule(new Task("one-time"), 1, TimeUnit.SECONDS);
  ```

- 以固定的每3秒执行：

  ```java
  // 2秒后开始执行定时任务，每3秒执行:
  ses.scheduleAtFixedRate(new Task("fixed-rate"), 2, 3, TimeUnit.SECONDS);
  ```

- 以固定的3秒为间隔执行：

  ```java
  // 2秒后开始执行定时任务，以3秒为间隔执行:
  ses.scheduleWithFixedDelay(new Task("fixed-delay"), 2, 3, TimeUnit.SECONDS);
  ```

后两种方式怎么看着没有区别，我们通过一张图就很容易理解：

![image-20210205102717343](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210205102717343.png)

Java标准库还提供了一个`java.util.Timer`类，这个类也可以定期执行任务，但是，一个`Timer`只会对应一个`Thread`，所以，一个`Timer`只能定期执行一个任务，多个定时任务必须启动多个`Timer`，而一个`ScheduledThreadPool`就可以调度多个定时任务，所以，我们完全可以用`ScheduledThreadPool`取代旧的`Timer`。

## 17、使用Future

在使用线程池执行任务时，我们提交的任务只需要实现 `Runnable` 接口就可以。

如果现在我们想要方法给返回值，就只能保存在变量中，再通过额外的方法读取，非常不便。所以，Java提供了一个 `Callable` 接口：

```java
public class Task implements Callable<String> {
    @Override
    public String call() throws Exception {
        return "";
    }
}
```

那么我们现在怎么才能获取到异步执行的结果呢，线程提交任务的 `ExecutorService.submit()` 方法，会返回一个 `Future` 实例，它代表了一个未来能获取结果的对象。

```java
ExecutorService executor = Executors.newFixedThreadPool(4); 
// 定义任务:
Callable<String> task = new Task();
// 提交任务并获得Future:
Future<String> future = executor.submit(task);
// 从Future获取异步执行返回的结果:
String result = future.get(); // 可能阻塞
```

> 在调用 `Future` 对象的 `get()` 方法时，如果异步任务完成，就直接获得到了结果。否则， `get()` 会阻塞，执行任务结束才返回结果。

`Future<V>` 接口定义的方法有：

- `get()`：获取结果（可能会等待）
- `get(long timeout, TimeUnit unit)`：获取结果，但只等待指定的时间；
- `cancel(boolean mayInterruptIfRunning)`：取消当前任务；
- `isDone()`：判断任务是否已完成。

## 18、使用CompletableFuture

使用 `Future` 可以获得异步执行的结果，但是在调用 `get()` 方法时，如果还没有执行结束，会被迫等待。

因此，在Java8中引入了 `CompletableFuture` ，它是对 `Future` 的改进，可以传入回调对象，当异步任务完成或者发生异常时，自动的调用回调对象的回调方法。

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // 创建异步执行任务:
        CompletableFuture<Double> cf = CompletableFuture.supplyAsync(new FetchPrice());
        // 如果执行成功:
        cf.thenAccept((result) -> {
            System.out.println("price: " + result);
        });
        // 如果执行异常:
        cf.exceptionally((e) -> {
            e.printStackTrace();
            System.out.println("发生异常");
            return null;
        });
        // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
        Thread.sleep(2000);
    }
}

class FetchPrice implements Supplier<Double> {
    @Override
    public Double get() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
        }
        if (Math.random() < 0.3) {
            throw new RuntimeException("fetch price failed!");
        }
        return 5 + Math.random() * 20;
    }
}
```

创建一个 `CompletableFuture` 是通过 `CompletableFuture.supplyAsync` 实现的，它需要一个实现了 `Supplier` 接口的对象。

接着，`CompletableFuture` 就会被提交给默认的线程池执行，我们需要定义的就是 `CompletableFuture` 执行完成或者异常时需要回调的实例。

在完成时，`CompletableFuture`会调用`Consumer`对象：

```java
public interface Consumer<T> {
    void accept(T t);
}
```

在异常时，`CompletableFuture`会调用`Function`对象：

```java
public interface Function<T, R> {
    R apply(T t);
}
```

这里我们都用lambda语法简化了代码。

`CompletableFuture` 的优点是：

- 异步任务结束时，自动回调方法；
- 异步任务出错时，自动回调方法；
- 主线程只用设置好回调，就不用关心异步任务的执行。

### 18.1 多个CompletableFuture串行执行



### 18.2 多个CompletableFuture并行执行



## 19、使用Fork/Join

Java7开始引入了一种新的 `Fork/Join` 线程池，它可以执行一种特殊的任务：把一个大任务拆成多个小任务并行执行。例如，我们需要计算一个超大数组的和，如果通过 `for` 循环执行需要的时间较多，这时，我们可以把数组拆分成两部分，用两个线程分别计算，最后加起来就是最终结果。当然，如果两部分还是太大，还可以继续拆分，用4个线程来执行。

![image-20210205103202438](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210205103202438.png)

这就是 `Fork/Join` 的原理：判断一个任务是否足够小，如果是，就直接计算，否则，就拆分为几个小任务分别计算。

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // 创建2000个随机数组成的数组:
        long[] array = new long[2000];
        long expectedSum = 0;
        for (int i = 0; i < array.length; i++) {
            array[i] = random();
            expectedSum += array[i];
        }
        System.out.println("Expected sum: " + expectedSum);

        // fork/join:
        ForkJoinTask<Long> task = new SumTask(array, 0, array.length);
        long startTime = System.currentTimeMillis();
        Long result = ForkJoinPool.commonPool().invoke(task);
        long endTime = System.currentTimeMillis();
        System.out.println("Fork/join sum: " + result + " in " + (endTime - startTime) + " ms.");
    }

    static Random random = new Random(0);

    static long random() {
        return random.nextInt(10000);
    }
}
/*核心代码，SumTask继承了RecursiveTask，并覆写了compute()方法。*/
class SumTask extends RecursiveTask<Long> {
    static final int THRESHOLD = 500;
    long[] array;
    int start;
    int end;

    SumTask(long[] array, int start, int end) {
        this.array = array;
        this.start = start;
        this.end = end;
    }


    @Override
    protected Long compute() {
        if (end - start <= THRESHOLD) {
            // 如果任务足够小,直接计算:
            long sum = 0;
            for (int i = start; i < end; i++) {
                sum += this.array[i];
                // 故意放慢计算速度:
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                }
            }
            return sum;
        }
        // 任务太大,一分为二:
        int middle = (end + start) / 2;
        System.out.println(String.format("split %d~%d ==> %d~%d, %d~%d", start, end, start, middle, middle, end));
        SumTask subtask1 = new SumTask(this.array, start, middle);
        SumTask subtask2 = new SumTask(this.array, middle, end);
        invokeAll(subtask1, subtask2);
        Long subresult1 = subtask1.join();
        Long subresult2 = subtask2.join();
        Long result = subresult1 + subresult2;
        System.out.println("result = " + subresult1 + " + " + subresult2 + " ==> " + result);
        return result;
    }
}
```

Fork/Join线程池在Java标准库中就有应用。Java标准库提供的`java.util.Arrays.parallelSort(array)`可以进行并行排序，它的原理就是内部通过Fork/Join对大数组分拆进行并行排序，在多核CPU上就可以大大提高排序的速度。

## 20、使用ThreadLocal

在Web应用中，每一个用户请求页面时，我们都会创建一个任务，类似：

```java
public void process(User user) {
    checkPermission();
    doWork();
    saveStatus();
    sendResponse();
}
```

然后让线程池去执行这些任务，但是我们想怎么在一个线程中传递状态呢？总不能把 `User` 实例传到每个方法中吧。在Java中，提供了一个特殊的 `ThreadLocal` ，它可以在一个线程中横跨多个方法传递需要的对象。而这个对象我们称为上下文（Context），它是一种状态，可以是用户信息等。

下面我们看一个例子：

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * @author : 
 * @date : 2020-03-12 22:28
 * @description :
 */

public class Main {

    public static void main(String[] args) throws Exception {
        ExecutorService es = Executors.newFixedThreadPool(3);
        String[] users = new String[] { "Bob", "Alice", "Tim", "Mike", "Lily", "Jack", "Bush" };
        for (String user : users) {
            es.submit(new Task(user));
        }
        es.shutdown();
    }

}

/*为了保证 ThreadLocal 的释放，可以通过实现 AutoCloseable接口配合 try(resource){...}*/
class UserContext implements AutoCloseable {

  	//ThreadLocal实例通常设置为静态字段
    private static final ThreadLocal<String> ctx = new ThreadLocal<>();

    public UserContext(String user) {
        ctx.set(user);
        System.out.printf("[%s] init user %s...\n", Thread.currentThread().getName(), UserContext.getUser());
    }

    public static String getUser() {
        return ctx.get();
    }

    @Override
    public void close() throws Exception {
        System.out.printf("[%s] cleanup for user %s...\n", Thread.currentThread().getName(), getUser());
        ctx.remove();
    }
}

class Task implements Runnable {

    private String name;

    public Task(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        try (UserContext uc = new UserContext(this.name)) {
            new Task1().process();
            new Task2().process();
            new Task3().process();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

class Task1 {
    public void process() {
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
        }
        System.out.printf("[%s] check user %s...\n", Thread.currentThread().getName(), UserContext.getUser());
    }
}

class Task2 {
    public void process() {
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
        }
        System.out.printf("[%s] %s registered ok.\n", Thread.currentThread().getName(), UserContext.getUser());
    }
}

class Task3 {
    public void process() {
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
        }
        System.out.printf("[%s] work of %s has done.\n", Thread.currentThread().getName(),
                UserContext.getUser());
    }
}
```

> 注意：普通的方法调用一定是在同一个线程中执行的，所以，`Task1`、`Task2`以及`Task3`中`UserContext.getUser()`获取的`User`对象是同一个实例。
>
> 我们可以把`ThreadLocal` 看成是一个全局的`Map<Thread, Object>`：每个线程在获取上下文时，通过`Thread` 自身最为key去获取。不同线程的 `ThreadLocal` 互不干扰。

## 21、Java内存模型

Java内存模型试图屏蔽各种硬件和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的内存访问效果。

Java的内存模型必须定义的足够严谨，保证在并发内存访问操作时不会出现歧义。也必须定义的足够宽松，才能更好地利用硬件的不同特性来获取更好地执行速度。

### 主内存与工作内存

所有的变量都存储在主内存中，每个线程还有自己的工作内存，工作内存存储在高速缓存或者寄存器中，线程的工作内存中保存了该线程使用的变量的主内存副本。

线程对变量的操作（读取、赋值等）必须在工作内存中进行，线程之间的变量值传递需要通过主内存来完成。

![image-20201228204735703](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201228204735703.png)

### 内存间交互操作

![image-20201228210248310](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201228210248310.png)

Java内存模型中定义了8种操作来完成主内存和工作内存之间的交互。

- lock（锁定）：作用于主内存的变量，标识一个变量的状态为线程独占。
- unlock（解锁）：作用于主内存的变量，释放处于锁定状态的变量。
- read（读取）：作用于主内存的变量，把一个变量的值从主内存传输到工作内存中。
- load（载入）：作用于工作内存中，在 read 之后执行，把 read 得到的值放入工作内存的变量副本中。
- use（使用）：作用于工作内存中，把工作内存中一个变量的值传递给执行引擎。
- assign（赋值）：作用于于工作内存中，把一个从执行引擎接收到的值赋给工作内存的变量。
- store（存储）：作用于工作内存中，把工作内存的一个变量的值传送到主内存中。
- write（写入）：作用于主内存中，在 store 之后执行，把 store 得到的值放入主内存的变量中。

### 内存模型如何实现三大特性

#### 原子性

Java 内存模型保证了 read、load、use、assign、store、write、lock 和 unlock 操作具有原子性，可以说对基本数据类型的访问、读写都具备原子性。如果需要更大程度上的原子性，则需要lock 和 unlock 的参与，反映到代码层面上就是同步块，即 synchronized关键字。

#### 可见性

可见性指当一个线程修改了共享变量的值，其它线程能够立即得知这个修改。

Java 内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的。

主要有三种实现可见性的方式：

- volatile
- synchronized，对一个变量执行 unlock 操作之前，必须把变量值同步回主内存。
- final，被 final 关键字修饰的字段在构造器中一旦初始化完成，并且没有发生 this 逃逸（其它线程通过 this 引用访问到初始化了一半的对象），那么其它线程就能看见 final 字段的值。

#### 有序性

有序性是指：在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了`指令重排序`和`工作内存和主内存同步延迟`。

在 Java 内存模型中，允许编译器和处理器对指令进行重排序，重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

volatile 关键字通过添加内存屏障的方式来禁止指令重排，即重排序时不能把后面的指令放到内存屏障之前。

也可以通过 synchronized 来保证有序性，它保证每个时刻只有一个线程执行同步代码，相当于是让线程顺序执行同步代码。

> 综上，synchronized 关键字是万能的，能同时保证上面3条特性。

## 22、volatile关键字

一个变量被定义成volatile之后，它将具备两项特性：

1. 保证此变量对所有线程的可见性。
2. 禁止指令重排序优化。

关于第一个特性的说明：

“可见性”是指一个线程修改了这个变量后，新值对于其他线程可以立即得知。

但这并不是说这个变量就是线程安全的。volatile变量在各个线程的工作内存中不存在一致性问题【虽然线程的工作内存中volatile变量的值有可能不一样，但因为每次使用之前都会刷新值，执行引擎看不到不一致的值，因此可以认为不存在一致性问题】，但是Java里面的运算符并不是原子操作，这就导致了volatile变量在并发下不是安全的。

```java
/**
 下面的代码中不能得到预期的 400000
*/
public class VolatileTest {

    public static volatile int race = 0;

    public static void increase() {
        race++;	
    }

    private static final int THREAD_COUNT = 20;

    public static void main(String[] args) {
        Thread[] threads = new Thread[THREAD_COUNT];
        for (int i = 0; i < THREAD_COUNT; i++) {
            threads[i] = new Thread(new Runnable() {
                public void run() {
                    for (int i1 = 0; i1 < 20000; i1++) {
                        increase();
                        System.out.println(race);
                    }
                }
            });
            threads[i].start();
        }

        while (Thread.activeCount() > 1) {
            Thread.yield();
        }

        System.out.println("=====" + race);
    }

}
```

>  由于volatile变量只能保证可见性，因此在不满足下面2个条件时，还是要通过加锁来保证原子性：
>
> - 运算结果并不依赖变量的当前值，或确保只有单一的线程去修改变量的值。
> -  变量不需要与其它的状态变量共同参与不变约束。
>
> ```java
> volatile boolean shutdownRequested;
> 
> public void shutdown() {
>     shutdownRequested = true;
> }
> 
> public void doWork() {
>     while(!shutdownRequested) {
>         //...
>     }
> }
> ```
>
> 

## 23、先行发生原则

先行发生原则是指在发生操作B之前，操作A产生的影响能够被操作B观察到。

时间先后顺序与先行发生原则之间基本没有因果关系，在衡量并发安全问题时，一切要以先行发生原则为准。

Java内存模型下，有一些天然的先行发生关系：

### 1. 单一线程原则

> Single Thread rule

在一个线程内，在程序前面的操作先行发生于后面的操作。

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/874b3ff7-7c5c-4e7a-b8ab-a82a3e038d20.png)

### 2. 管程锁定规则

> Monitor Lock Rule

一个 unlock 操作先行发生于后面对同一个锁的 lock 操作。（这里是指同一个锁）

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/8996a537-7c4a-4ec8-a3b7-7ef1798eae26.png)

### 3. volatile 变量规则

> Volatile Variable Rule

对一个 volatile 变量的写操作先行发生于后面对这个变量的读操作。

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/942f33c9-8ad9-4987-836f-007de4c21de0.png)

### 4. 线程启动规则

> Thread Start Rule

Thread 对象的 start() 方法调用先行发生于此线程的每一个动作。

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/6270c216-7ec0-4db7-94de-0003bce37cd2.png)

### 5. 线程终止规则

> Thread Termination Rule

线程中的所有操作都先行发生于对此线程的终止检测。可以通过 `Thread::join()` 方法是否结束、`Thread::isAlive()` 的返回值等手段检测线程是否已经终止执行。

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/233f8d89-31d7-413f-9c02-042f19c46ba1.png)

### 6. 线程中断规则

> Thread Interruption Rule

对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过 interrupted() 方法检测到是否有中断发生。

### 7. 对象终结规则

> Finalizer Rule

一个对象的初始化完成（构造函数执行结束）先行发生于它的 finalize() 方法的开始。

### 8. 传递性

> Transitivity

如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C。

## 24、线程安全

### 安全程度

按照线程安全的 `安全程度` 由强到弱来排序，可以将Java中各种操作共享的数据分为5类。

#### 1. 不可变

不可变（Immutable）的对象一定是线程安全的，不需要再采取任何的线程安全保障措施。

不可变的类型：

- final 关键字修饰的基本数据类型
- String
- 枚举类型
- Number 部分子类，如 Long 和 Double 等数值包装类型，BigInteger 和 BigDecimal 等大数据类型。但同为 Number 的原子类 AtomicInteger 和 AtomicLong 则是可变的。

#### 2. 绝对线程安全

Java API中标注自己是线程安全的类，大多数都不是绝对线程安全的。这些类的方法，如果不在方法调用端做一些同步措施，仍然可能会出现线程不安全。

#### 3. 相对线程安全

我们平时讲的线程安全就是相对线程安全的，例如`Vector`、`HashTable`。它保证了对对象的单次操作时线程安全的。

#### 4. 线程兼容

线程兼容是说虽然对象本身不是线程安全的，但可以在调用端通过同步手段来保证在并发环境下的安全使用。例如`ArrayList` 、`HashMap` 。

#### 5. 线程对立

线程对立是指不管调用端是否采取了同步措施，都不能在并发环境下安全的使用代码。

### 线程安全的实现

#### 1. 互斥同步

同步是指多个线程并发访问共享数据时，保证共享数据只能被一个线程使用。

互斥是实现同步的一种手段。

实现方法有synchronized 和 ReentrantLock。【二者的区别和选择 看上面的第10小节】

#### 2. 非阻塞同步

随着硬件指令集的发展，我们可以使用基于冲突检测的乐观并发策略：先进行操作，如果没有其它线程争用共享数据，那操作就成功了，否则采取补偿措施（不断地重试，直到成功为止）。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。

乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成。硬件支持的原子性操作最典型的是：比较并交换（Compare-and-Swap，CAS）。

> CAS存在的逻辑漏洞：ABA问题。
>
> 如果一个变量初次读取的时候是 A 值，它的值被改成了 B，后来又被改回为 A，那 CAS 操作就会误认为它从来没有被改变过。
>
> J.U.C 包提供了一个带有标记的原子引用类 AtomicStampedReference 来解决这个问题，它可以通过控制变量值的版本来保证 CAS 的正确性。大部分情况下 ABA 问题不会影响程序并发的正确性，如果需要解决 ABA 问题，改用传统的互斥同步可能会比原子类更高效。

#### 3. 无同步方案

如果一个方法本来就不涉及共享数据，那它自然就不需要任何同步措施去保证线程安全。

- 可重入代码：

  这种代码也叫做纯代码（Pure Code），可以在代码执行的任何时刻中断它，转而去执行另外一段代码（包括递归调用它本身），而在控制权返回后，原来的程序不会出现任何错误。

  可重入代码有一些共同的特征，例如不依赖存储在堆上的数据和公用的系统资源、用到的状态量都由参数中传入、不调用非可重入的方法等。

- 线程本地存储（Thread Local Storage）：

  如果一段代码中所需要的数据必须与其他代码共享，那就看看这些共享数据的代码是否能保证在同一个线程中执行。如果能保证，我们就可以把共享数据的可见范围限制在同一个线程之内，这样，无须同步也能保证线程之间不出现数据争用的问题。

## 25、锁优化

### 自旋锁与自适应自旋

互斥同步进入阻塞状态的开销都很大（挂起线程和恢复线程的操作都需要转入内核态中完成），应该尽量避免。在许多应用中，共享数据的锁定状态只会持续很短的一段时间。自旋锁的思想是让后面请求锁的那个线程执行一个忙循环（自旋），如果在这段时间内能获得锁，就可以避免进入阻塞状态。

自旋等待虽然减少了线程切换的时间，但还是要占用处理器时间的，因此==它只适用于锁被占用时间很短的情况==。基于这个原因，应该对自旋等待的时间有一定的限制，如果超过了某个设定值（默认是10次），就应该使用传统的方式去挂起线程。

在JDK1.6中引入了`自适应的自旋锁`。自适应意味着自旋的次数不再固定了，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。

### 锁消除

锁消除是指对于被检测出不可能存在共享数据竞争的锁进行消除。

锁消除主要是通过逃逸分析来支持，如果堆上的共享数据不可能逃逸出去被其它线程访问到，那么就可以把它们当成栈上数据对待，认为它们是线程私有的，自然就不需要同步加锁。

对于一些看起来没有加锁的代码，其实隐式的加了很多锁。例如下面的字符串拼接代码就隐式加了锁：

```java
public static String concatString(String s1, String s2, String s3) {
    return s1 + s2 + s3;
}
```

String 是一个不可变的类，编译器会对 String 的拼接自动优化。在 JDK 1.5 之前，会转化为 StringBuffer 对象的连续 append() 操作：

```java
public static String concatString(String s1, String s2, String s3) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    sb.append(s3);
    return sb.toString();
}
```

每个 append() 方法中都有一个同步块。虚拟机观察变量 sb，经过逃逸分析后会发现它的动态作用域被限制在 concatString() 方法内部。也就是说，sb 的所有引用永远不会逃逸到 concatString() 方法之外，其他线程无法访问到它，因此可以进行消除。

在解释执行时这里仍然会加锁，但是在即时编译后，就会忽略同步措施。

### 锁粗化

如果一系列的连续操作都对同一个对象反复加锁和解锁，频繁的加锁操作就会导致性能损耗。

上面的代码中连续的 append() 方法就属于这类情况。如果虚拟机探测到由这样的一串零碎的操作都对同一个对象加锁，将会把加锁的范围扩展（粗化）到整个操作序列的外部。对于上面的代码就是扩展到第一个 append() 操作之前直至最后一个 append() 操作之后，这样只需要加锁一次就可以了。

### Monitor

每个Java对象都可以关联一个 Monitor对象，如果使用 synchronized 给对象上锁（重量级锁）之后，该对象头的Mark Work就会设置成指向Monitor对象的指针。

![image-20210208193748662](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210208193748662.png)

- 刚开始Monitor中 Owner 为null

- 当Thread-2执行synchronized(obj)就会将Monitor的所有者Owner置为Thread-2，Monitor中只能有一个Owner
- 在Thread-2上锁的过程中，如果Thread-3，Thread-4，Thread-5也来执行synchronized(obj)，就会进入EntryList BLOCKED
- Thread-2执行完同步代码块的内容，然后唤醒EntryList 中等待的线程来竞争锁，竞争时是非公平的
- 图中WaitSet 中的Thread-0，Thread-1是之前获得过锁，但条件不满足进入WAITING状态的线程

### 无锁 VS 偏向锁 VS 轻量级锁 VS 重量级锁

JDK 1.6 引入了偏向锁和轻量级锁，从而让锁拥有了四个状态，级别从低到高依次是：无锁状态（unlocked）、偏向锁状态（biasble）、轻量级锁状态（lightweight locked）和重量级锁状态（inflated）。锁状态只能升级不能降级。

以下是HotSpot虚拟机中对象头的内存布局，这些数据称为 `Mark Word`。

![image-20210208172203158](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210208172203158.png)

![image-20201229223152544](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201229223152544.png)



### 偏向锁

轻量级锁在没有竞争时，每次重入仍然需要执行CAS操作，

在JDK6中引入了偏向锁来优化：==只有第一次使用 CAS 将线程 ID 设置到对象的 Mark Word 头，之后发现 这个线程 ID 是自己的就表示没有竞争，不用重新 CAS。以后只要不发生竞争，这个对象就归该线程所有。==

偏向锁只有遇到其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁，线程不会主动释放偏向锁。根据锁对象目前是否处于锁定状态决定是否撤销偏向（偏向模式设置为“0”）。撤销偏向锁后恢复到无锁（标志位为“01”）或轻量级锁（标志位为“00”）的状态。状态转化及对象 `Mark Word` 的关系如下：

![image-20201230155742007](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201230155742007.png)

偏向锁的工作过程：

1. 访问Mark Word中偏向锁的标识是否设置成1，锁标志位是否为01——确认为可偏向状态。
2. 如果为可偏向状态，则测试线程ID是否指向当前线程，如果是，进入步骤（5），否则进入步骤（3）。
3. 如果线程ID并未指向当前线程，则通过CAS操作竞争锁。如果竞争成功，则将Mark Word中线程ID设置为当前线程ID，然后执行（5）；如果竞争失败，执行（4）。
4. 如果CAS获取偏向锁失败，则表示有竞争。当到达全局安全点（safepoint）时获得偏向锁的线程被挂起，偏向锁升级为轻量级锁，然后被阻塞在安全点的线程继续往下执行同步代码。
5. 执行同步代码。

偏向锁在JDK 6及以后的JVM里是默认启用的。可以通过JVM参数关闭偏向锁：`-XX:-UseBiasedLocking=false`，关闭之后程序默认会进入轻量级锁状态。

### 轻量级锁

轻量级锁的使用场景：如果一个对象虽然有多线程要加锁，但加锁的时间是错开的（也就是没有竞争），那么可以使用轻量级锁来优化。

“轻量级”是相对于使用操作系统互斥量来实现的传统锁而言的。但是，轻量级锁并不是用来代替重量级锁的，它的本意是在没有多线程竞争的前提下，减少传统的重量级锁使用产生的性能消耗。

> 当锁处于偏向锁时，如果被另外的线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，从而提高性能。

轻量级锁的工作过程：

1. 在代码进入同步块的时候，如果同步对象锁状态为无锁状态（锁标志位为“01”状态，是否为偏向锁为“0”），虚拟机首先将在当前线程的栈帧中建立一个名为锁记录（`Lock Record`）的空间，用于存储锁定对象的 `Mark Word`。

2. 将`Lock Record`里的`owner`指针指向锁对象，并使用CAS操作尝试将对象的`Mark Word`更新为指向`Lock Record`的指针，将`Mark Work` 存入`Lock Record`。

   ![image-20210208202104091](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210208202104091.png)

3. 如果这个更新动作成功了，那么这个线程就拥有了该对象的锁，并且对象`Mark Word`的锁标志位设置为“00”，表示此对象处于轻量级锁定状态。

4. 如果轻量级锁的更新操作失败了，虚拟机首先会检查对象的`Mark Word`是否指向当前线程的栈帧，如果是就说明当前线程已经拥有了这个对象的锁，那就可以直接进入同步块继续执行，否则说明多个线程竞争锁，执行步骤5。

5. 若当前只有一个等待线程，则该线程通过自旋进行等待。但是当自旋超过一定的次数，或者一个线程在持有锁，一个在自旋，又有第三个来访时，轻量级锁升级为重量级锁。

### 重量级锁

升级为重量级锁时，锁标志的状态值变为“10”，此时`Mark Word`中存储的是指向重量级锁的指针，此时等待锁的线程都会进入阻塞状态。

- 当轻量级锁膨胀为重量级锁时，需要为Object对象申请 Minitor锁，让Object对象指向重量级锁地址。然后其他的线程进入Monitor的EntryList BLOCKED

![image-20210208221649639](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210208221649639.png)

- 当 Thread-0 退出同步块解锁时，使用 cas 将 `Mark Word` 的值恢复给对象头，失败【因为此时已经升级为重量级锁，标记为10】。这时会进入重量级解锁流程，即按照 Monitor 地址找到 Monitor 对象，设置 Owner 为 null，然后唤醒 EntryList 中 BLOCKED 线程。

![image-20201230161613623](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201230161613623.png)

参考：[不可不说的Java“锁”事](https://tech.meituan.com/2018/11/15/java-lock.html)

[Synchronized低层优化（偏向锁、轻量级锁）](https://www.cnblogs.com/paddix/p/5405678.html)

## 26、Semaphore

`Semaphore` 用于限制能同时访问共享资源的线程上限。

```java
public class SemaphoreTest {

    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);

        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try {
                    semaphore.acquire();
                    System.out.println("start...");
                    // 休眠1秒
                    TimeUnit.SECONDS.sleep(1);
                    System.out.println("end...");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                }
            }, "Thread-" + i).start();
        }
    }
}

/**
start...
start...
start...
end...
end...
end...
start...
start...
start...
end...
end...
end...
start...
start...
start...
end...
end...
end...
start...
end...
*/
```

## 27、CountDownLatch

用来进行线程同步协作，等待所有线程完成倒计时。

- `await()` 被调用后，当前线程会进入等待状态，在计数器归0后会被唤醒。
- `countDown()` 每次调用会让计数器的值减1

```java
public class CountDownLatchTest {

    public static void main(String[] args) {
        CountDownLatch countDownLatch = new CountDownLatch(3);
        ExecutorService service = Executors.newFixedThreadPool(4);

        service.submit(() -> {
            System.out.println("t4 run");
            try {
                countDownLatch.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("t4 end");
        });
        service.submit(() -> {
            System.out.println("t1 run");
            // 休眠1秒
            try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
            countDownLatch.countDown();
            System.out.println("t1 end");
        });
        service.submit(() -> {
            System.out.println("t2 run");
            // 休眠2秒
            try { TimeUnit.SECONDS.sleep(2); } catch (InterruptedException e) { e.printStackTrace(); }
            countDownLatch.countDown();
            System.out.println("t2 end");
        });
        service.submit(() -> {
            System.out.println("t3 run");
            // 休眠3秒
            try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
            countDownLatch.countDown();
            System.out.println("t3 end");
        });
        service.shutdown();
    }

}

/**
t4 run
t1 run
t2 run
t3 run
t1 end
t2 end
t3 end
t4 end
*/
```

## 28、CyclicBarrier

用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行。

和 CountdownLatch 相似，都是通过维护计数器来实现的。线程执行 `await()` 方法之后计数器会减 1，并进行等待，直到计数器为 0，所有调用 await() 方法而在等待的线程才能继续执行。

`CyclicBarrier` 和 `CountdownLatch` 的一个区别是，CyclicBarrier 的计数器通过调用 `reset()` 方法可以循环使用，所以它才叫做循环屏障。

`CyclicBarrier`有两个构造函数，其中 `parties` 指示计数器的初始值，`barrierAction` 在所有线程都到达屏障的时候会执行一次。

```java
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}

public CyclicBarrier(int parties) {
    this(parties, null);
}
```

```java
public class CyclicBarrierTest {
    public static void main(String[] args) {
        CyclicBarrier barrier = new CyclicBarrier(2, () -> {
            System.out.println("thread-1,thread-2 finish...");
        });
        ExecutorService service = Executors.newFixedThreadPool(2);
        for (int i = 1; i <= 2; i++) {
            int k = i;
            service.submit(() -> {
                System.out.println("thread-" + k +" start...");
                // 休眠k秒
                try {
                    TimeUnit.SECONDS.sleep(k);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                try {
                    barrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
                System.out.println("thread-" + k +" end...");
            });
        }
        service.shutdown();
    }
}

/**
thread-1 start...
thread-2 start...
thread-1,thread-2 finish...
thread-2 end...
thread-1 end...
*/
```



# 十、XML

## 1、XML简介

XML是可扩展标记语言（Extensible Markup Language）的缩写，它是一种数据表示格式，可以描述十分复杂的数据结构常用于传输和存储数据。

一个XML文档大概是长这样：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE note SYSTEM "book.dtd">
<book id="1">
    <name>Java核心技术</name>
    <author>Cay S. Horstmann</author>
    <isbn lang="CN">1234567</isbn>
    <tags>
        <tag>Java</tag>
        <tag>Network</tag>
    </tags>
    <pubDate/>
</book>
```

> xml默认使用UTF-8编码；
>
> xml内容经常通过网络作为消息传说。

### 1.1 XML的结构

xml有着固定的结构，第一行一定是 `<?xml version="1.0" ?>` ，可以加上可选的编码。

`<!DOCTYPE note SYSTEM "book.dtd">` 声明的是文档定义类型（DTD: Document Type Definition），是可选的。

下面的才是xml的文档内容。需要注意的是一个xml文档有且仅有一个根元素。

当内容中出现了特殊符号时，需要转义，因为xml文档中已经使用 `<` 、 `>`、 `''` 等做标识符。

| 字符 | 表示     |
| :--- | :------- |
| <    | `&lt;`   |
| >    | `&gt;`   |
| &    | `&amp;`  |
| "    | `&quot;` |
| '    | `&apos;` |

例如内容为 `Java<tm>` 时应该写成：

```xml
<name>Java&lt;tm&gt;</name>
```

## 2、解析XML

XML是一种树形结构的文档，它有着两种标准的解析API：

- DOM：一次性读取XML，并在内存中表示为树形结构；
- SAX：以流的形式读取XML，使用事件回调。

以下面的xml为例（book.xml）：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<book id="1" category="computer">
    <name>Java核心技术</name>
    <author>Cay S. Horstmann</author>
    <isbn lang="CN">1234567</isbn>
    <tags>
        <tag>Java</tag>
        <tag>Network</tag>
    </tags>
    <pubDate/>
</book>
```



### 2.1 使用DOM

`book.xml` 解析为DOM结构如下：

![](images/QQ20200313-134634.png)

Java提供了DOM API来解析xml，使用了下面的对象来表示xml的内容：

- Document：代表整个xml文档；

- Element：代表一个xml元素；

  xml元素指的是从 `开始标签` 到 `结束标签` 的部分。一个元素可以包括：

  - 其他元素
  - 文本
  - 属性（下面的Attribute）

- Attribute：代表一个元素的某个属性。

![](images/QQ20200313-135847.png)

使用DOM API解析XML文档的代码如下：

```java
public class XMLUtil {

    public static void main(String args[]) throws IOException, SAXException, ParserConfigurationException {
        parseXML();
    }

    public static void parseXML() throws ParserConfigurationException, IOException, SAXException {
        InputStream input = XMLUtil.class.getResourceAsStream("/book.xml");
        DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
        DocumentBuilder db = dbf.newDocumentBuilder();
        Document doc = db.parse(input);
        printNode(doc, 0);
    }

  	//遍历以读取指定元素的值：
    static void printNode(Node n, int indent) {
        for (int i = 0; i < indent; i++) {
            System.out.print(' ');
        }
        switch (n.getNodeType()) {
            case Node.DOCUMENT_NODE:
                System.out.println("Document: " + n.getNodeName());
                break;
            case Node.ELEMENT_NODE:
                System.out.println("Element: " + n.getNodeName());
                break;
            case Node.TEXT_NODE:
                System.out.println("Text: " + n.getNodeName() + " = " + n.getNodeValue());
                break;
            case Node.ATTRIBUTE_NODE:
                System.out.println("Attr: " + n.getNodeName() + " = " + n.getNodeValue());
                break;
            case Node.CDATA_SECTION_NODE:
                System.out.println("CDATA: " + n.getNodeName() + " = " + n.getNodeValue());
                break;
            case Node.COMMENT_NODE:
                System.out.println("Comment: " + n.getNodeName() + " = " + n.getNodeValue());
                break;
            default:
                System.out.println("NodeType: " + n.getNodeType() + ", NodeName: " + n.getNodeName());
        }
        for (Node child = n.getFirstChild(); child != null; child = child.getNextSibling()) {
            printNode(child, indent + 1);
        }
    }

}
```

> 其中的 `DocumentBuilder.parse()` 用于解析一个xml，它可以接受InputStream、File、URL，会返回一个Document对象，这个对象代表了整个xml文档的属性结构

输出的机构如下：

```java
//输出结果
Document: #document
 Element: book
  Text: #text = 
    
  Element: name
   Text: #text = Java核心技术
  Text: #text = //输出这个是因为在xml中，元素 `name` 和元素 `author` 存在换行，解析器把当成了text处理
    
  Element: author
   Text: #text = Cay S. Horstmann
  Text: #text = 
    
  Element: isbn
   Text: #text = 1234567
  Text: #text = 
    
  Element: tags
   Text: #text = 
        
   Element: tag
    Text: #text = Java
   Text: #text = 
        
   Element: tag
    Text: #text = Network
   Text: #text = 
    
  Text: #text = 
    
  Element: pubDate
   Text: #text = 2020-03-13
  Text: #text = 
```



### 2.2 使用SAX

使用DOM解析的优点是简单省事，但它的主要缺点是如果文件过大，占用内存太大。

针对内存太大的问题，就有了另外一种解析xml的方法是SAX（Simple API for XML）。它是一种基于流的解析方法，边读取XML边解析，并以事件回调的方法让调用者获取数据。也正是因为一边读取一边解析，所以不论XML文件多大，占用的内存都很小。

SAX解析会触发一系列的事件：

- startDocument：开始读取XML文档；
- startElement：读取到了一个元素，例如`<book>`；
- characters：读取到了字符；
- endElement：读取到了一个结束的元素，例如`</book>`；
- endDocument：读取XML文档结束。

如果用SAX API解析XML，其Java代码如下：

```java
public class XMLUtil {

    public static void main(String args[]) throws Exception {
        parseXML2();
    }
  
    public static void parseXML2() throws Exception {
        InputStream input = XMLUtil.class.getResourceAsStream("/book.xml");
        SAXParserFactory spf = SAXParserFactory.newInstance();
        SAXParser saxParser = spf.newSAXParser();
        saxParser.parse(input, new MyHandler());
    }
}
class MyHandler extends DefaultHandler {
    public void startDocument() throws SAXException {
        print("start document");
    }

    public void endDocument() throws SAXException {
        print("end document");
    }

    public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
        print("start element:", localName, qName);
    }

    public void endElement(String uri, String localName, String qName) throws SAXException {
        print("end element:", localName, qName);
    }

    public void characters(char[] ch, int start, int length) throws SAXException {
        print("characters:", new String(ch, start, length));
    }

    public void error(SAXParseException e) throws SAXException {
        print("error:", e);
    }

    void print(Object... objs) {
        for (Object obj : objs) {
            System.out.print(obj);
            System.out.print(" ");
        }
        System.out.println();
    }
}
```

> 关于`startElement` 中的参数是什么意思？
>
> ```java
> /**
>      * Receive notification of the start of an element.
>      *
>      * <p>By default, do nothing.  Application writers may override this
>      * method in a subclass to take specific actions at the start of
>      * each element (such as allocating a new tree node or writing
>      * output to a file).</p>
>      *
>      * @param uri The Namespace URI, or the empty string if the
>      *        element has no Namespace URI or if Namespace
>      *        processing is not being performed.
>      * @param localName The local name (without prefix), or the
>      *        empty string if Namespace processing is not being
>      *        performed.
>      * @param qName The qualified name (with prefix), or the
>      *        empty string if qualified names are not available.
>      * @param attributes The attributes attached to the element.  If
>      *        there are no attributes, it shall be an empty
>      *        Attributes object.
>      * @exception org.xml.sax.SAXException Any SAX exception, possibly
>      *            wrapping another exception.
>      * @see org.xml.sax.ContentHandler#startElement
>      */
> ```
>
> 

### 2.3 转为JavaBean

DOM和SAX两种解析XML的标准接口，使用起来都不直观。

幸运的，我们可以使用 `Jackson` 这个开源库把XML转化为JavaBean。

首选需要导入依赖包：

```xml
<dependency>
  <groupId>com.fasterxml.jackson.dataformat</groupId>
  <artifactId>jackson-dataformat-yaml</artifactId>
  <version>2.10.3</version>
</dependency>

<dependency>
  <groupId>org.codehaus.woodstox</groupId>
  <artifactId>woodstox-core-asl</artifactId>
  <version>4.4.1</version>
</dependency>
```

创建一个JavaBean `Book.java` ：

```java
/*Book.java*/
public class Book {
    private long id;
    private String name;
    private String category;
    private String author;
    private String isbn;
    private List<String> tags;
    private String pubDate;
  //省略getter和setter
  //省略toString
}
```

解析测试：

```java
@Test
public void m2() throws IOException {
  InputStream input = Main.class.getResourceAsStream("/book.xml");
  JacksonXmlModule module = new JacksonXmlModule();
  XmlMapper mapper = new XmlMapper(module);
  Book book = mapper.readValue(input, Book.class);

  System.out.println(book);
}
```

```java
//输出结果
Book{id=1, name='Java核心技术', category='computer', author='Cay S. Horstmann', isbn='1234567', tags=[Java, Network], pubDate='2020-03-13'}
```

# 十一、函数式编程

函数式编程的一个特点就是，允许把函数本身作为参数传入另一个函数，还允许返回一个函数。

从 Java8 开始支持函数式编程。

## 1、Lambda表达式

Java的方法分为实例方法，例如 `Integer` 定义的 `equals()` 方法：

```java
public final class Integer {
    boolean equals(Object o) {
        ...
    }
}
```

以及静态方法，例如 `Integer` 定义的 `parseInt()` 方法：

```java
public final class Integer {
    public static int parseInt(String s) {
        ...
    }
}
```

上面的方法，本质上都相当于过程式语言的函数。只不过Java的实例方法隐含的传入了一个 `this` 变量。

函数式编程是把函数作为基本元算单元，函数可以作为变量，可以接受函数，还可以返回函数。

在Java程序中，我们经常遇到一些但方法接口，即一个接口只定义一个方法：

- Comparator
- Runnable
- Callable

以 `Comparator` 为例，从 Java8 开始，我们可以使用 Lambda表达式替换单方法接口：

```java
@Test
public void m0() {
  String[] array = new String[] { "Apple", "Orange", "Banana", "Lemon" };
  Arrays.sort(array, (s1, s2) -> {
    return s1.compareTo(s2);
  });
  System.out.println(String.join(", ", array));
}
```

和 `JavaScript` 的箭头函数一样，在方法中只有一句时，可以省略 `{}` ：

```java
Arrays.sort(array, (s1, s2) -> s1.compareTo(s2));
```

> 在Lambda 表达式中，它只需要写出方法定义，参数为（s1, s2），参数的类型可以省略，因为编译器可以自动推断出 `String` 类型。
>
> 返回值的类型也是由编译器自动推断。

### 1.1 FunctionalInterface

只定义了单方法的接口称之为 `FunctionalInterface` ，用注解 `@FunctionalInterface` 标记。例如， `Callable` 接口：

```java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

在接口`Comparator`中：

```
@FunctionalInterface
public interface Comparator<T> {

    int compare(T o1, T o2);

    boolean equals(Object obj);

    default Comparator<T> reversed() {
        return Collections.reverseOrder(this);
    }

    default Comparator<T> thenComparing(Comparator<? super T> other) {
        ...
    }
    ...
}
```

虽然 `Comparator` 接口有很多方法，但只有一个抽象方法 `int compare(T o1, T o2)` ，其他方法都是`default` 和`static` 方法。`boolean equals(object obj)`是 `Object` 定义的方法，不算在接口方法内。

## 2、方法引用 

使用 Lambda表达式，我们可以不必编写 `FunctionalInterface` 接口的实现类，从而简化了代码。

当然，除了 Lambda 表达式，还可以直接传入方法引用，例如：

```java
public class LambdaTest {

    @Test
    public void m1() {
        String[] array = new String[] { "Apple", "Orange", "Banana", "Lemon" };
        Arrays.sort(array, LambdaTest::cmp);
        System.out.println(String.join(", ", array));
    }

    static int cmp(String s1, String s2) {
        return s1.compareTo(s2);
    }


}
```

上述代码在 `Arrays.sort()` 中传入了静态方法 `cmp` 的引用，用 `LambdaTest::cmp` 表示。

方法引用，就是说某个方法签名和接口一样，就可以直接传入方法引用。

ps：方法签名只看参数类型和返回类型，不看方法名称，也不看类的继承关系。

### 2.1 构造方法引用

如果要把一个`String` 数组转化为 `Person` 数组，在以前，我们可能会：

```java
@Test
public void m2() {
  Stream<String> stream = Stream.of("Bob", "Alice", "Tim");
  List<Person> list = new ArrayList<>();
  stream.forEach(s -> {
    list.add(new Person(s));
  });
  System.out.println(list);
}
```

现在我们可以引用 `Person` 的构造方法来实现 `String` 到 `Person` 的转化：

```java
public class Main { 
    public static void main(String[] args) {
        List<String> names = List.of("Bob", "Alice", "Tim");
        List<Person> persons = names.stream().map(Person::new).collect(Collectors.toList());
        System.out.println(persons);
    }
}

class Person {
    String name;
    public Person(String name) {
        this.name = name;
    }
    public String toString() {
        return "Person:" + this.name;
    }
}
```



## 3、使用Stream

从 Java8 开始，引入了一种全新的流式API：Stream API。位于 `java.util.stream` 包中。

这个 `stream` 和 `java.io` 中的 `InputStram` 和 `OutputStream` 不一样，它代表的是任意Java对象的序列。

|      | java.io                  | java.util.stream           |
| :--- | :----------------------- | -------------------------- |
| 存储 | 顺序读写的`byte`或`char` | 顺序输出的任意Java对象实例 |
| 用途 | 序列化至文件或网络       | 内存计算／业务逻辑         |



`java.util.stream` 和 `List` 也不一样，`List` 存储的每个元素都已经在内存中存储，而 `stream` 输出的对象可能并没有存储在内存中，而是实时计算得到的，且是惰性计算的。

简单来说，`List` 就是一个个实实在在的元素，这些元素也已经存储在内存中，用户可以用它来操作其中的元素（例如，遍历、排序等）。而 `stream` 可能就根本没有分配内存。下面，看一个例子：

如果想用 `List` 表示全体的自然数，这是不可能的，因为自然数是无穷的，但内存是有限的。

如果我们使用 `stram` 就可以做到，如下：

```java
Stream<BigInteger> naturals = createNaturalStream(); // 全体自然数
```

> 上面的 createNaturalStram() 方法没有实现。

也可以对 `Stream` 计算，例如，对每个自然数做一个平方：

```java
Stream<BigInteger> naturals = createNaturalStream(); // 全体自然数
Stream<BigInteger> streamNxN = naturals.map(n -> n.multiply(n)); // 全体自然数的平方
```

上面的 `streamNxN` 也有无限多个元素，如果要打印它，可以用 `limit()` 方法截取前100个元素，最后用 `forEach()` 处理每个元素。

```java
Stream<BigInteger> naturals = createNaturalStream();
naturals.map(n -> n.multiply(n)) // 1, 4, 9, 16, 25...
        .limit(100)
        .forEach(System.out::println);
```

> `Stream` 惰性计算的特点：一个 `Stream` 转换为另一个时，实际上只存储了转化规则，并不会有任何的计算。例如，上面的例子中，只有在调用 `forEach` 确实需要输出元素时，才会进行计算。  



### 3.1 创建Stream

#### Stream.of()

使用 `Stream.of()` 创建虽然没有实质性用途，但在测试时很方便。

```java
@Test
public void m1() {
  Stream<String> stream = Stream.of("A", "B", "C", "D");
  // forEach()方法相当于内部循环调用，
  // 可传入符合Consumer接口的void accept(T t)的方法引用：
  stream.forEach(System.out::println);
}
```

#### 基于数组或Collection

可以基于一个数组或者 `Collection` 创建`Stream` ，这样`Stream` 在输出时的元素也就是数组或 `Collection` 的元素。

```java
@Test
public void m1() {
  //数组变成 `Stream` 使用 `Arrays.stream()` 方法
  Stream<String> stream1 = Arrays.stream(new String[] { "A", "B", "C" });
  //对于 `Collection`（List/Set/Queue等），直接调用 `stream()` 方法即可。
  Stream<String> stream2 = List.of("X", "Y", "Z").stream();
  stream1.forEach(System.out::println);
  stream2.forEach(System.out::println);
}
```

> 上述创建 `Stream` 的方法都是把一个现有的序列变成`Stream` ，它的元素都是固定的。

#### 基于Supplier 

创建 `Stream` 还可以通过 `Stream.generate()` 方法，它需要传入的是一个 `Supplier` 对象：

```java
Stream<String> s = Stream.generate(Supplier<String> sp);
```

这时 `Stream` 保存的不是具体的元素，而是一种规则，在需要产生一个元素时，`Stream` 自己回去调用 `Supplier.get()` 方法。

例子，通过`Stream` 不断的产生自然数：

```java
public class StreamTest {
    @Test
    public void m1() {
        Stream<Integer> natural = Stream.generate(new NaturalSupplier());
        natural.limit(10).forEach(System.out::println);
    }
}

class NaturalSupplier implements Supplier<Integer> {
    int n = 0;
    @Override
    public Integer get() {
        return ++n;
    }
}
```

即使`int` 的范围有限，但如果用 `List` 存储，也会占用巨大的内存，而使用 `Stream` 时，因为只保存计算规则，所以几乎不占用空间。

在调用 `forEach()` 或者 `count()` 进行最终求值前，一定要把 `Stream` 的无限序列变成有限序列，否则会因为不能完成这个计算进入死循环。

#### 其他方法

创建 `Stream` 的第三种方法是通过一些API提供的接口，直接获得 `Stream`。

例如，`Files` 类的 `lines()` 方法把一个文件变成 `Stream` ，每个元素代表文件的一行内容：

```java
try (Stream<String> lines = Files.lines(Paths.get("/path/to/file.txt"))) {
    ...
}
```

> 在需要对文本文件按行遍历时，该方法十分有用。

正则表达式的`Pattern`对象有一个`splitAsStream()`方法，可以直接把一个长字符串分割成`Stream`序列而不是数组：

```java
Pattern p = Pattern.compile("\\s+");
Stream<String> s = p.splitAsStream("The quick brown fox jumps over the lazy dog");
s.forEach(System.out::println);
```

#### 基本类型

因为Java的泛型不支持基本类型，所以不能使用 `Stream<int>` 这种类型。为了方便，Java标准库提供了 `IntStream` 、`LongStream` 、`DoubleStream` 这3种使用基本类型的`Stream` ，它们的使用方法和范型`Stream`没有大的区别。

### 3.2 使用map 

Java中的`map` 、`filter`、 `reduce` 类似于`JavaScript` 中[高阶函数](https://blog.csdn.net/weixin_40971059/article/details/104409588)的用法。

类似的用法，可以写出下面的例子：

```java
@Test
public void m2() {
  List.of(1, 2, 3, 4)
    .stream()
    .map(n -> n * n)	//求平方
    .forEach(System.out::println); // 打印
}
```

### 3.3 使用filter

```java
@Test
public void m2() {
  IntStream.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
    .filter(n -> n % 2 != 0)
    .forEach(System.out::println);
}
```

### 3.4 使用reduce

`map()`和`filter()`都是`Stream`的转换方法，而`Stream.reduce()`则是`Stream`的一个聚合方法。

```java
@Test
public void m2() {
  int n = IntStream.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
    .reduce(0, (x, y) -> x + y);
  System.out.println(n);
}
```

> 上面的代码如果去掉初始值，会返回一个 `Optional<Integer>` ：
>
> ```java
> Optional<Integer> opt = stream.reduce((acc, n) -> acc + n);
> if (opt.isPresent) {
>  System.out.println(opt.get());
> }
> ```
>
> 这是因为 `Stream` 的元素有可能是0个，这样就没法调用 `reduce()` 方法，因此返回 `Optional` 对象，需要怕断结果是否存在。

> 对 `Stream` 的操作分为两类：
>
> - 转换操作：把一个 `Stream` 转化为 另一个 `Stream` ，例如 `map()` 和 `filter()` ，
> - 聚合操作：会对`Stream` 的每个元素进行计算，得到一个确定的结果，例如`reduce()` ，这类操作会触发计算。

### 3.5 输出集合

#### 输出为List

因为需要把 `Stream` 的元素保存到集合，而集合保存的都是确定的 Java对象，所以把 `Stream` 变成 `List` 是一个聚合操作。

```java
@Test
public void m2() {
  Stream<String> stream = Stream.of("Apple", "", null, "Pear", "  ", "Orange");
  List<String> lists = 
    stream.filter(s -> s != null && !s.trim().isEmpty()).collect(Collectors.toList());
  System.out.println(lists);
}
```

把`Stream`的每个元素收集到`List`的方法是调用`collect()`并传入`Collectors.toList()`对象，它实际上是一个`Collector`实例，通过类似`reduce()`的操作，把每个元素添加到一个收集器中（实际上是`ArrayList`）。

类似的，`collect(Collectors.toSet())`可以把`Stream`的每个元素收集到`Set`中。

#### 输出为数组

```java
@Test
public void m2() {
  Stream<String> stream = Stream.of("Apple", "Pear", "Orange");
  String[] array = stream.toArray(String[]::new);
  System.out.println(String.join(", ", array));
}
```

> 传入的“构造方法”是`String[]::new`，它的签名实际上是`IntFunction`定义的`String[] apply(int)`，即传入`int`参数，获得`String[]`数组的返回值

#### 输出为Map

对于 `Stream` 的元素输出到 `Map` ，需要分别把元素映射为key和value：

```java
@Test
public void m2() {
  Stream<String> stream = Stream.of("APPL:Apple", "MSFT:Microsoft");
  Map<String, String> map = stream
    .collect(Collectors.toMap(
      // 把元素s映射为key:
      s -> s.substring(0, s.indexOf(':')),
      // 把元素s映射为value:
      s -> s.substring(s.indexOf(':') + 1)));
  System.out.println(map);
}
```

#### 分组输出

`Stream` 还有一个强大的功能就是可以按组输出。

```java
@Test
public void m2() {
  Stream<String> stream = 
    Stream.of("Apple", "Banana", "Blackberry", "Coconut", "Avocado", "Cherry", "Apricots");
  	Map<String, List<String>> groups = stream
    	.collect(Collectors.groupingBy(s -> s.substring(0, 1), Collectors.toList()));
  	System.out.println(groups);
}
```

```java
//输出结果
{A=[Apple, Avocado, Apricots], B=[Banana, Blackberry], C=[Coconut, Cherry]}
```

在上面使用到的 `Collectors.groupinigBy()` 方法，需要提供两个函数：

- 第一个是分组的key，`s -> s.substring(0, 1)` 表示只要首字母相同的 `String` 分到一组。
- 第二个是分组的value，这里直接输出为 `List`。



 




























































































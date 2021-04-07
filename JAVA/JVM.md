

# 虚拟机与JAVA虚拟机

虚拟机，就是一台虚拟的计算机。它是一款软件，用来执行一系列虚拟计算机指令。大体上，虚拟机可以分为 `系统虚拟机` 和 `程序虚拟机`。

- `Visual Box`、`VMware`就属于`系统虚拟机`，它们==完全是对物理计算机的仿真==，提供了一个可运行完整操作系统的软件平台。
- `程序虚拟机`的典型代表就是 Java 虚拟机，它==专门为执行单个计算机程序而设计==，在Java虚拟机中执行的指令我们称为 Java字节码指令。

## 1、Java虚拟机

- Java虚拟机是一台执行Java字节码的虚拟计算机，它拥有独立的运行机制，其运行的Java字节码也未必由Java语言编译而成。
- JVM平台的各种语言可以共享Java虚拟机带来的跨平台性、优秀的垃圾回器，以及可靠的即时编译器。
- ==Java技术的核心就是Java虚拟机==（JVM，Java Virtual Machine），因为所有的Java程序都运行在Java虚拟机内部。

- `作用`：==Java虚拟机就是二进制字节码的运行环境==，负责装载字节码到其内部，解释/编译为对应平台上的机器指令执行。每一条Java指令，Java虚拟机规范中都有详细定义，如怎么取操作数，怎么处理操作数，处理结果放在哪里。
- `特点`：一次编译，到处运行；自动内存管理；自动垃圾回收功能

![image-20201005214356253](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201005214356253.png)

## 2、JVM的生命周期

1. 虚拟机的启动

   Java虚拟机的启动是通过引导类加载器(bootstrap class loader)创建一个初始类(initial class)来完成的，这个类是由虚拟机的具体实现指定的。

2. 虚拟机的执行

   - 一个运行中的Java虚拟机有着一个清晰的任务：执行Java程序。
   - 程序开始执行时他才运行，程序结束时他就停止。
   - ==执行一个所谓的Java程序的时候，真真正正在执行的是一个叫做Java虚拟机的进程==。

3. 虚拟机的退出

   有如下的几种情况:

   - 程序正常执行结束
   - 程序在执行过程中遇到了异常或错误而异常终止
   - 由于操作系统出现错误而导致Java虚拟机进程终止
   - 某线程调用 `Runtime` 类或 `system` 类的exit方法，或`Runtime`类的halt方法，并且Java安全管理器也允许这次exit或halt操作。


# 类的加载

## 类加载器子系统

![image-20201011113118793](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201011113118793.png)

- 类加载器子系统负责从文件系统或者网络中加载class文件，class文件在文件开头有特定的文件标识。
- `classLoader` 只负责class文件的加载，至于它是否可以运行，则由 `Execution Engine` 决定。
- 加载的类信息存放于一块称为方法区的内存空间。除了类的信息外，方法区中还会存放运行时常量池信息，可能还包括字符串字面量和数字常量（这部分常量信息是class文件中常量池部分的内存映射）

## 类的加载过程

![image-20201011113807051](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201011113807051.png)

1. 加载：

   1. 通过一个类的全限定名获取定义此类的二进制字节流 => 并没有指明二进制字节流要从哪里获取，可以从ZIP压缩包（JAR、WAR）、网络、运行时自动生成、从加密文件中获取等。
   2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
   3. `在内存中生成一个代表这个类的java.lang.class对象`，作为方法区这个类
      的各种数据的访问入口

2. 链接

   ![image-20201011114258803](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201011114258803.png)

3. 初始化

   ![image-20201011120428828](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201011120428828.png)


## 类的加载器分类

把类加载阶段中的==通过一个类的全限定名来获取描述该类的二进制字节流”== 这个动作放到虚拟机外部去实现，以让应用程序自己决定如何获取所需的类。实现这个动作的代码就是`类加载器`。

JVM 支持两种类型的 类加载器，分别为 `引导类加载器(Booostrap ClassLoader)` 和 `自定义类加载器(User-Defined ClassLoader)`。

Java 虚拟机规范==将所有派生于抽象类 ClassLoader 的类加载器都划分为自定义类加载器==。

在程序中常见的3个类加载器：

![image-20201011122023372](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201011122023372.png)

```java
public class ClassLoaderTest {

    public static void main(String[] args) {
        /** 系统类加载器 */
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        System.out.println(systemClassLoader); //sun.misc.Launcher$AppClassLoader@18b4aac2

        /** 扩展类加载器 */
        ClassLoader exClassLoader = systemClassLoader.getParent();
        System.out.println(exClassLoader); //sun.misc.Launcher$ExtClassLoader@677327b6

        /** 引导类加载器 --- 获取不到 */
        ClassLoader bsClassLoader = exClassLoader.getParent();
        System.out.println(bsClassLoader); //null

        /** 对于用户自定义类来说，默认使用系统类加载器进行加载 */
        ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
        System.out.println(classLoader); //sun.misc.Launcher$AppClassLoader@18b4aac2

        /** String类使用引导类加载器进行加载 --- Java的核心类库都是*/
        ClassLoader classLoader1 = String.class.getClassLoader();
        System.out.println(classLoader1); //null

    }

}
```

### 引导类加载器

- 这个类加载器使用 `C/C++` 语言实现，嵌套在 JVM 内部
- 用来加载 Java 的核心库（），用于提供 JVM 自身需要的类
- 并不继承`java.lang.classLoader`，没有父加载器
- 加载扩展类和应用类加载器，并指定为他们的父类加载器。
- 出于安全考虑，`Bootstrap` 启动类加载器只加载包名为` java、javax、sun`等开头的类

### 扩展类加载器

- `Java语言编写`，由 `sun.misc.Launcher$ExtClassLoader` 实现。
- 派生于classLoader类
- 父类加载器为启动类加载器
- 从 `java.ext.dirs` 系统属性所指定的目录中加载类库，或从JDK的安装目录的 `jre/lib/ext` 子目录（扩展目录）下加载类库。==如果用户创建的JAR放在此目录下，也会自动由扩展类加载器加载==。

### 系统类加载器

- java语言编写，由 `sun.misc.Launcher$AppclassLoader` 实现
- 派生于classLoader类
- 父类加载器为扩展类加载器
- 它负责加载环境变量classpath或系统属性 java.class.path 指定路径下的类库
- ==该类加载是程序中默认的类加载器==。一般来说，Java应用的类都是由它来完成加载
- 通过`ClassLoader#getSystemClassLoader()`方法可以获取到该类加载器

## 双亲委派机制

Java虚拟机对class文件采用的是`按需加载`的方式，也就是说当需要使用该类时才会将它的class文件加载到内存生成class对象。而且加载某个类的class文件时，Java虚拟机采用的是`双亲委派模式`，即把请求交由父类处理，它是一种任务委派模式。

### 工作原理

![image-20201011202053566](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201011202053566.png)

### 优势

- 避免了类的重复加载
- 保护程序安全，防止核心的 API 被篡改
  - 自定义类：`java.lang.String`

## 沙箱安全机制

![image-20201011203134433](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201011203134433.png)

> 在JVM中判断两个class类是否为同一个类的两个必要条件：
>
> - 类的包名、类名一样
> - 加载这个类的 `ClassLoader` 必须相同

## 类的主动使用与被动使用

![](https://gitee.com/yanjundong97/picBed/raw/master/images/20201029095748.png)



















# 执行引擎

## 概述

执行引擎是Java虚拟机核心的组成部分之一。

“虚拟机”是一个相对于“物理机”的概念，这两种机器都有代码执行能力，其区别是物理机的执行引擎是直接建立在处理器、缓存、指令集和操作系统层面上的，而==虚拟机的执行引擎则是由软件自行实现的==，因此可以不受物理条件制约地定制指令集与执行引擎的结构体系，==能够执行那些不被硬件直接支持的指令集格式==。

JVM的任务是负责==装载字节码到其内部==，但字节码不能直接运行在操作系统上，因为字节码指令并不等价于本地机器指令，它内部包含的仅仅只是一些能够被JVM所识别的字节码指令、符号表，以及其他辅助信息。

那么，如果想要一个Java程序运行起来，执行引擎（Execution Engine）的工作就是==将字节码指令解释/编译为对应平台的本地机器指令才可以==。

## 即时编译器 

==既然HotSpot VM中已经内置JIT编译器了，那么为什么还需要再使用解释器来“拖累”程序的执行性能呢?==

当Java虛拟器启动时，解释器可以首先发挥作用，而不必等待 `即时编译器（JIT,just in time）` 全部编译完成后再执行，这样可以省去许多不必要的编译时间。随着时间的推移，编译器把越来越多的代码编译成本地代码，开始发挥作用，获得更高的执行效率。

> 当程序启动后，解释器可以马上发挥作用，省去编译的时间，立即执行。
> 编译器要想发挥作用，要先把代码编译成本地代码，需要一定的执行时间。但编译为本地代码后，执行效率高。

# 内存结构

![image-20201005224109716](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201005224109716.png)

![image-20201005224318647](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201005224318647.png)

## 一、程序计数器

- 是程序控制流的指示器，程序的分支、循环、跳转、异常处理、线程恢复都需要依赖于程序计数器。
- `线程私有` 
- 如果线程正在执行的是一个Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是本地（Native）方法，这个计数器值则应为空（Undefined）。
- 没有OOM（`OutOfMemoryError`）
- 没有GC

## 二、Java虚拟机栈

 栈是运行时的单位，堆是存储的单位。即：栈解决程序的运行问题，即程序如何执行，或者说如何处理数据。堆解决的是数据存储的问题，即数据怎么放、放在哪儿。

### 1、基本内容

- 每个线程在创建时都会创建一个虚拟机栈，其内部保存着一个个的栈帧（`Stack Frame`），对应着一次次的Java方法调用。

- 主管Java程序的运行，它保存方法的局部变量、部分结果，并参与方法的调用和返回。

- 没有GC

- 可能会有`StackOverflowError` 或者 `OutOfMemoryError` 

  - 如果线程请求的栈容量大于虚拟机所允许的最大容量，将抛出 `StackOverflowError` 异常；如果Java虚拟机栈容量可以动态扩展，当栈扩展时无法申请到足够的内存会抛出 `OutOfMemoryError`异常。 

  - > HotSpot虚拟机的栈容量是不可以动态扩展的，所以HotSpot虚拟机的虚拟机栈不存在 `OOM` 。但是如果在申请时就失败，仍会OOM异常。

### 2、栈的存储单位

- 每个线程都有自己的栈，栈中的数据都是以栈帧（`Stack Frame`）的格式存在
- 在这个线程上正在执行的每个方法都各自对应一个栈帧
- 栈帧是一个内存区块，是一个数据集，维系着方法执行过程中的各种数据信息。

### 3、栈运行原理

- 在一个活动线程中，一个时间点，只会有一个活动的栈帧，即只有当前正在执行的栈帧是有效的，这个栈帧被称为`当前栈帧`。
- 执行引擎执行的所有字节码只针对当前栈帧来进行操作。
- 如果在该方法中调用了其他方法，对应新的栈帧就会被创建出来，放在栈顶。

### 4、栈帧的内部结构

每个栈帧中存储着：

- ==局部变量表（`Local Variables`）==

- ==操作数栈（`Operand Stack`）【或表达式栈】==

- 动态链接（`Dynamic Linking`）【或指向运行时常量池的方法引用】

- 方法返回地址（`Return Address`）【或方法正常退出或异常退出的定义】

- 一些附加信息

  ![](https://gitee.com/yanjundong97/picBed/raw/master/images/20201029152149.png)

#### 4.1 局部变量表

- 定义为一个数字数组，主要==用于存储方法参数和定义在方法体内的局部变量，这些数据类型包括基本数据类型、对象引用，以及`returnAddress`类型。==
- 由于局部变量表示建立在线程的栈上，是线程私有的数据，因此不存在数据安全问题。

- 局部变量表所需的容量大小是在编译期就确定下来的，并保存在Code属性的 `maximum local variables` 数据项中。在方法运行期间是不会改变局部变量表的大小。
- 局部变量表，==最基本的存储单元式Slot（变量槽）==
- 在局部变量中，32位以为的类型只占用一个Slot（包括`returnAddress`类型），64位的类型占用两个Slot。
  - byte、short、char在存储前转化为int，boolean也被转化为int。

- 如果当前帧是由构造方法或者实例方法创建的，那么该对象引用 `this` 将会存放在index为0的slot处，其余的参数按照参数表顺序继续排列。 

#### 4.2 变量的分类

 按照数据类型分：

- 基本数据类型
- 引用数据类型

按照在类中声明的位置分：

- 成员变量：在使用前，都经历过默认初始化赋值
  - 类变量：linking的prepare阶段：给类变量默认赋值 ---> initial阶段：给类变量显式赋值即静态代码块赋值 
  - 实例变量：随着对象的创建，会在堆空间中分配实例变量空间，并进行默认赋值。
- 局部变量：在使用前，必须要进行显示赋值！否则，编译不通过。

```java
public class LocalVariable {

    private int a;
    private String str;


    public void method() {
        int b;
        String s1;
        //System.out.println(b); //错误！！！Variable 'b' might not have been initialized
        //System.out.println(s1); //错误！！！Variable 's1' might not have been initialized
        
        System.out.println(a); //0
        System.out.println(str); //null
    }

    public static void main(String[] args) {
        LocalVariable localVariable = new LocalVariable();
        localVariable.method();
    }

}
```

> 局部变量表中的变量也是重要的垃圾回收根节点，只要被局部变量表中直接或间接引用的对象都不会被回收。

#### 4.3 操作数栈

操作数栈，在方法执行过程中，根据字节码指令，往栈中写入数据或提取数据。

![](https://gitee.com/yanjundong97/picBed/raw/master/images/20201029174824.png)

- 操作数栈，==主要用于保存计算过程中的中间结果，同时作为计算过程中变量临时的存储空间==。
- 操作数栈是JVM执行引擎的一个工作区，当一个方法刚开始执行的时候，一个新的栈帧被创建，此时这个方法的操作数栈是空的。
- 每一个操作数栈都会拥有一个明确的栈深度用于存储数值，其所需的最大深度在编译期就定义好了，保存在方法的code属性中，为max stack的值。

#### 4.4 动态链接（==指向运行时常量池的方法引用==）

![](https://gitee.com/yanjundong97/picBed/raw/master/images/20201029192528.png)

![image-20200820103808645](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200820103808645.png)

#### 4.5 方法的调用

在JVM中，将符号引用转换为调用方法的直接引用与方法的绑定机制有关。

- 静态链接：当一个字节码文件被加载进JVM内部时，如果==被调用的目标方法在编译期可知==，且运行期保持不变。这种情况下，将调用方法的符号引用转换为直接引用的过程称为`静态链接`。
- 动态链接：如果被调用的方法==在编译期无法被确定下来==，只能在程序运行时将调用方法的符号引用转化为直接引用，由于这种转换过程具备动态性，称为`动态链接`。

> 非虚方法---静态链接：
>
> - 如果方法在编译期就确定了具体的调用版本，这个版本在运行时是不可变的。这样的方法称为`非虚方法`
> - `静态方法`、`私有方法`、`final方法`、`实例构造器`、`父类方法`都是非虚方法。
> - 其他方法称为虚方法。

#### 4.6 方法返回地址

- 存放调用该方法的pc寄存器的值
- 无论通过哪种方式退出，在方法退出后都返回到该方法被调用的位置。
- 方法正常退出时，==调用者的pc计数器的值作为返回地址，即调用该方法的指令的下一条指令的地址==。而通过异常退出的，返回地址是要通过异常表来确定，栈帧中一般不会保存这部分信息。 

## 三、本地方法栈

- Java虚拟机栈用于管理Java方法的调用，而本地方法栈用于管理本地方法的调用。
- 本地方法栈，也是线程私有的。
- ==当某个线程调用一个本地方法时，它就进入一个全新的并且不再受虚拟机限制的世界。它和虚拟机拥有着同样的权限。==
- 在 `HotSpot JVM` 中，直接==将本地方法栈和虚拟机栈合二为一==。

## 四、堆

### 1、概述

### 2、MinorGC、MajorGC和FullGC

JVM在进行GC时，并非每次都对上面的三个内存（新生代、老年代；方法区）区域一起回收，大部分时候回收的都是 `新生代`。

针对 `HotSpot VM` 的实现，它里面的 GC 按照回收区域又分为两种类型：一种是部分收集（`Partial GC`），一种是整堆收集（`Full GC`）

- 部分收集：不是完整收集整个Java堆的垃圾收集。其中又分为：
  - 新生代收集（`Minor GC  / Young GC`）：只是新生代的垃圾收集
  - 老年代收集（`Major GC / Old GC`）：只是老年代的垃圾收集。
    - 目前，只有`CMS GC` 会有单独收集老年代的行为。
    - 注意：很多时候`Major GC` 和 `Full GC` 混淆使用，需要具体分辨是`老年代回收`还是`整堆回收`
  - 混合回收（`Mixed GC`）：收集整个新生代以及部分老年代的垃圾收集。
    - 目前，只有 `G1 GC` 会有这种行为。

- 整堆收集（`Full GC`）：收集整个Java堆和方法区的垃圾收集。

#### 2.1 年轻代GC(Minor GC)触发机制:

- 当年轻代空间不足时，就会触发MinorGC，这里的年轻代满指的是Eden代满，==Survivor满不会引发GC==。(每次 Minor GC会清理年轻代的内存。)
- 因为Java 对象大多都具备`朝生夕灭`的特性，所以Minor GC非常频繁，一般回收速度也比较快。
- Minor GC会引发STW，暂停其它用户的线程，等垃圾回收结束，用户线程才恢复运行。

#### 2.2 老年代GC (Major GC/Full GC) 触发机制:

- 指发生在老年代的GC，对象从老年代消失时，我们说`Major GC` 或`Fu11 GC`发生了。
- 出现了Major GC， 经常会伴随至少一次的Minor GC (但非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行Major GC的策略选择过程)。
  - 也就是在老年代空间不足时，会先尝试触发Minor GC 。如果之后空间还不足，则触发Major GC
- Major GC的速度一般会比Minor GC慢10倍以上，STW的时间更长。
- 如果Major GC后，内存还不足，就报`00M` （OutOfMemoryError）了。

### 3、堆是分配对象存储的唯一选择吗？

> 随着JIT编译期的发展与 `逃逸分析技术` 逐渐成熟，`栈上分配`、`标量替换优化技术` 将会导致一些微妙的变化，所有的对象都分配到堆上也变得不那么“绝对”了。

如果经过逃逸分析（Escape Analysis）后发现，==一个对象并没有逃逸出方法的话，那么就可能被优化成栈上分配==。这样就无需在堆上分配内存，也无需进行垃圾回收了。这也就是`堆外存储技术`。

#### 3.1 逃逸分析概述

- 如何将堆上的对象分配到栈，需要使用逃逸分析手段。
- 这是一种可以有效减少Java程序中`同步负载`和`内存堆分配压力`的跨函数全局数据流分析算法。
- 通过逃逸分析，Java Hotspot编译器能够分析出一个新的对象的引用的使用范围，从而决定是否要将这个对象分配到堆上。
- 逃逸分析的基本行为就是分析`对象动态作用域`:
  - 当一个对象在方法中被定义后，对象只在方法内部使用，则认为没有发生逃逸。
  - 当一个对象在方法中被定义后，它被外部方法所引用，则认为发生逃逸。例如作为调用参数传递到其他地方中。

#### 3.2 逃逸分析：代码优化

如果一个对象不会发生逃逸，则可能为这个变量进行一些高效地优化:

- 栈上分配：如果一个对象不会发生逃逸，可以让这个对象在栈上分配内存，对象所占用的内存会随着栈帧的出栈而销毁。减少了GC的时间消耗。
- 同步消除：如果一个对象不会发生`线程逃逸`，那么对这个变量实施的操作可以不考虑同步。
- 标量替换：如果一个对象不会被外部访问，并且这个对象可以被拆解（由`聚合量`到`标量`），那么该对象就不需要在堆上创建，而是直接在栈上创建这个对象的成员变量。

## 五、方法区

### 1、栈、堆、方法区的交互关系

#### 运行时数据区结构图

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ截图20200819154548.png)

 

#### 栈、堆、方法区的交互关系

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ截图20200819154704.png)

 

### 2、方法区概述

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ截图20200819161058.png)

- 到了 JDK 8，就完全放弃了`永久代`的概念，改为与JRockit、J9一样在`本地内存` （并非Java虚拟机的内存）中实现的元空间（Metaspace）来代替。

  ![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ截图20200819161856.png)

- 元空间的本质与永久代类似，都是对JVM规范中方法区的实现。==两者最大的区别是：元空间不在虚拟机设置的内存中，而是使用本地内存==。

- 如果方法区无法满足新的内存分配需要时，将会抛出OOM异常。

### 3、方法区的内部结构

#### 方法区存储内存

用于存储已被虚拟机加载的**类型信息、常量、静态变量、即时编译器编译后的代码缓存**等数据。

### 4、方法区的演进细节

| 版本         | 细节                                                         |
| ------------ | ------------------------------------------------------------ |
| jdk1.6及之前 | 有永久代，`静态变量存放在永久代中`                           |
| jdk1.7       | 有永久代，但已经逐渐“去永久代”，`字符串常量池、静态变量移除，保存到堆中`。 |
| jdk1.8及之后 | 无永久代，类型信息、字段、方法、常量保存在本地内存的元空间，但`字符串常量池、静态变量仍在堆中` |

> 只有HotSpot才有永久代的说法，这样做的好处是：GC能够像管理堆一样管理这部分内存，减少工作。但是带来的缺点就是Java应用很容易遇到内存溢出的问题（如果是放在元空间中，是使用本地内存，不再使用虚拟机的内存，容量更大）。

#### 字符串常量池为什么要调整位置？

jdk7中将StringTable放到了堆空间中。因为永久代的回收效率很低，在full gc的时候才会触发。而full gc是老年代的空间不足、永久代不足时才会触发。这就导致StringTable回收效率不高。

而我们开发中会有大量的字符串被创建，回收效率低，导致永久代内存不足。放到堆里，能及时回收内存。

### 5、方法区的垃圾收集

方法区垃圾收集的“性价比”通常是比较低的。

方法区的垃圾收集主要回收两部分内容：

- 常量池中废弃的常量，与Java堆中对象的回收类似，如果没有任何地方引用一个常量，那么这个常量就可以被系统清理出常量池。
- 不再使用的类型，要判断一个类型是否属于“不再被使用的类”则是比较苛刻的，需要同时满足3个条件。

### 6、总结

![image-20200820103808645](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200820103808645.png)

### 7、常见面试题

![image-20200820104131223](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200820104131223.png)

![image-20200820104047927](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200820104047927.png)

 

# HotSpot虚拟机对象

## 对象的创建

![image-20200820113840759](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200820113840759.png)

1. 首先检查类是否被加载、解析和初始化过。如果没有，就必须先执行相应的类加载过程。
2. 为新生对象分配内存（对象所需内存的大小在类加载完成后就可以完全确定），为对象分配空间就是从Java堆中划分一块空间。
   - 堆中内存是`绝对规整`的：所有被使用过的内存都被放在一边，空闲的内存被放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针向空闲空间方向挪动一段与对象大小相等的距离，这种分配方式称为“`指针碰撞`”
   - 如果堆中的内存`不是规整`的：虚拟机就必须维护一个列表，记录上哪些内存块是可用的，在需要时划分一块足够大的空间，这种分配方式称为“`空闲列表`”。
   - ==说明==：选择哪种分配方式由Java堆是否规整决定，而Java堆是否规整又由所采用的垃圾收集器是否带有空间压缩整理（Compact）的能力决定。

3. 并发情况下，即使是`指针碰撞` 这种简单的方式，==对象创建也并不是线程安全的==。解决该问题的两种方案：
   - 采用CAS（Compare And Set）配上失败重试的方式来保证操作的原子性。
   - 每个线程在Java堆中分配一小块内存，称为`本地线程分配缓冲（Thread Local Allocation Buffer, TLAB）`。可以通过-XX：+/-UseTLAB参数来设定。

4. 将分配到的内存空间（不包括对象头）都初始化为零值。这步操作保证了对象的实例字段在不赋初始值时就可以直接使用。
5. 设置对象的对象头
6. 执行Class文件中的<init>()方法。

## 对象的内存布局

![image-20200820163025927](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200820163025927.png)

![image-20200820162031375](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200820162031375.png)

## 对象的访问定位

Java程序是通过reference数据来操作堆上的具体对象，主流的对象访问方式有两种：`句柄访问`和`直接指针`。

### 句柄访问

在Java堆中专门划分一块内存作为`句柄池` ，句柄中包含了对象实例数据与类型数据各自具体的地址信息。

最大好处就是reference中存储的是稳定句柄地址，在对象被移动（垃圾收集时移动对象是非常普遍的行为）时只会改变句柄中的实例数据指针，而reference本身不需要被修改。

![image-20200820164807387](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200820164807387.png)

### 直接指针

Java堆中对象的内存布局就必须考虑如何放置访问类型数据的相关信息，reference中存储的直接就是对象地址，如果只是访问对象本身的话，就不需要多一次间接访问的开销。

HotSpot主要采用这种对象访问方式。

好处：

- 速度很快，减少了一次指针定位的时间开销。

![image-20200820164613043](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200820164613043.png)

# StringTable

## String的基本特性

字符串常量池是不会存储相同内容的字符串的。

- String的String Pool 是一个固定大小的Hashtable，默认值长度是1009。
- 使用`-XX:StringTableSize` 可以设置StringTable的长度
- 在JDK6中StringTable的长度默认是==1009==。
- 在JDK7中，StringTable的长度默认是==60013==。
- JDK8后，设置StringTable的长度时，1009是可以设置的最小值。

## String内存位置的变化

看上面`方法区的演进细节`  以及 `字符串常量池为什么要调整位置？`

## 字符串拼接操作

1. 常量与常量的拼接结果是在常量池，原理是编译器优化。

2. 常量池中不会存在相同结果的常量

3. 只要其中有一个是变量，结果就是在堆中。变量拼接的原理是StringBuilder。

   ```java
   //从生成的字节码中，也可以看出这一点	
   //下面测试测试的部分字节码如下：
     0 ldc #2 <Hadoop>
     2 astore_1
     3 ldc #3 <Hdfs>
     5 astore_2
     6 ldc #4 <HadoopHdfs>
     8 astore_3
     9 ldc #4 <HadoopHdfs>	//编译器优化：在编译阶段，常量与常量就拼接成HadoopHdfs
    11 astore 4
    13 new #5 <java/lang/StringBuilder>	//使用StringBuiler进行 s1 + "Hdfs"
    16 dup
    17 invokespecial #6 <java/lang/StringBuilder.<init>>
    20 aload_1
    21 invokevirtual #7 <java/lang/StringBuilder.append>
    24 ldc #3 <Hdfs>
    26 invokevirtual #7 <java/lang/StringBuilder.append>
    29 invokevirtual #8 <java/lang/StringBuilder.toString>
    32 astore 5
    34 new #5 <java/lang/StringBuilder>
    37 dup
    38 invokespecial #6 <java/lang/StringBuilder.<init>>
    41 ldc #2 <Hadoop>
    43 invokevirtual #7 <java/lang/StringBuilder.append>
    46 aload_2
    47 invokevirtual #7 <java/lang/StringBuilder.append>
    50 invokevirtual #8 <java/lang/StringBuilder.toString>
    53 astore 6
    55 new #5 <java/lang/StringBuilder>
    58 dup
    59 invokespecial #6 <java/lang/StringBuilder.<init>>
    62 aload_1
    63 invokevirtual #7 <java/lang/StringBuilder.append>
    66 aload_2
    67 invokevirtual #7 <java/lang/StringBuilder.append>
    70 invokevirtual #8 <java/lang/StringBuilder.toString>
    73 astore 7
   
   ```

4. 如果拼接结果调用了 `intern()` 方法，则主动将常量池中还没有的字符串对象放入池中，并返回此对象地址。

```java
@Test
public void test1() {
    String s1 = "Hadoop";
    String s2 = "Hdfs";

    String s3 = "HadoopHdfs";
    String s4 = "Hadoop" + "Hdfs";//编译器优化
    //如果拼接符号的前后出现了变量，则相当于在堆空间中new String()， 
    //具体的内容为拼接的结果: HadoopHdfs
    String s5 = s1 + "Hdfs";
    String s6 = "Hadoop" + s2;
    String s7 = s1 + s2;

    System.out.println(s3 == s4);//true
    System.out.println(s3 == s5);//false
    System.out.println(s3 == s6);//false
    System.out.println(s3 == s7);//false
    System.out.println(s5 == s6);//false
    System.out.println(s5 == s7);//false
    System.out.println(s6 == s7);//false

    //intern():判断字符串常量池中是否存在HadoopHdfs值，如果存在，则返回常量池中HadoopHdfs的地址;
    //如果字符串常量池中不存在HadoopHdfs，则在常量池中加载一份HadoopHdfs, 并返回次对象的地址。
    String s8 = s6.intern();
    System.out.println(s3 == s8);//true
}
```

## `intern()` 的使用

### new String() 创建了几个对象

`new String("abc")` 会创建2个对象：

- new 关键字在堆空间创建的对象
- 字符串常量池中的对象，字节码指令：ldc

```java
@Test
public void test3() {
    String s2 = new String("abc");
}

//上面代码的字节码为：
 0 new #15 <java/lang/String>	//new 一个String对象
 3 dup
 4 ldc #16 <abc>	//在字符串常量池中放入abc
 6 invokespecial #17 <java/lang/String.<init>>
 9 astore_1
10 return
```



`new String("a") + new String("b")` 会创建几个对象呢？

1. 对象1：new StringBuilder
2. 对象2：new String("a")
3. 对象3：常量池中的a
4. 对象4：new String("b")
5. 对象5：常量池中的b

> StringBuilder的`toString()` 方法：
>
> 尽管该方法体中就是new 了一个 String对象，但是和普通的 new 一个String对象不同，它只创建了一个对象。
>
> ```java
> @Override
> public String toString() {
>  // Create a copy, don't share the array
>  return new String(value, 0, count);
> }
> ```
>
> ```java
> 0 new #80 <java/lang/String>
> 3 dup
> 4 aload_0
> 5 getfield #234 <java/lang/StringBuilder.value>
> 8 iconst_0
> 9 aload_0
> 10 getfield #233 <java/lang/StringBuilder.count>
> 13 invokespecial #291 <java/lang/String.<init>>
> 16 areturn
> ```
>
> 在字符串常量池中，并没有生成。



 ### 死坑题

```java
public class StringIntern {

    public static void main(String[] args) {
        String s1 = new String("1");
        String intern1 = s1.intern();//调用此方法之前，字符串常量池中已经存在了"1"
        String s2 = "1";
        System.out.println(s1 == s2);   //jdk6：false；jdk7/8：false
        System.out.println(intern1 == s2);  //true

        //s3变量记录的地址为：new String("12")
        String s3 = new String("1") + new String("2");
        //执行完上一行代码以后，字符串常量池中，是否存在"12"呢？答案：不存在！！
        //在字符串常量池中生成"12"。
        //如何理解：jdk6:创建了一个新的对象"12",也就有新的地址。
                 //jdk7:此时常量中并没有创建"12",而是创建一个指向堆空间中new String("12")的地址,					//目的就是为了节省内存空间
        String intern2 = s3.intern();
        //s4变量记录的地址：使用的是上一行代码代码执行时，在常量池中生成的"12"的地址
        String s4 = "12";
        System.out.println(s3 == s4);   //jdk6：false；jdk7/8：true
        System.out.println(intern2 == s4);  //true
    }
}
```

## 总结

总结String的`intern()`的使用：

- Jdk1.6中，将这个字符串对象尝试放入串池。
  - 如果串池中有，则并不会放入。返回已有的串池中的对象的地址
  - 如果没有，会把==此对象复制一份==，放入串池，并返回串池中的对象地址
- Jdk1.7起，将这个字符串对象尝试放入串池。
  - 如果串池中有，则并不会放入。返回已有的串池中的对象的地址
  - 如果没有，则会把==对象的引用地址复制一份==， 放入串池，并返回串池中的引用地址

 

# GC

## 一、垃圾回收概述

 ### 什么是垃圾？

 垃圾就是指在`运行程序中没有任何指针指向的对象`，这个对象就是需要被回收的垃圾。

如果不及时对内存中的垃圾进行清理，那么，这些垃圾对象所占的内存空间会一直保留到应用程序结束，被保留的空间无法被其他对象使用。甚至可能导致==内存溢出==。

### Java自动内存管理

- 自动内存管理，不需要参与内存的分配与回收，这样可以==降低内存的泄露和内存溢出的风险==。
- 自动内存管理机制，可以==更专注于业务开发==。
- 垃圾回收的区域：

![image-20200822214505611](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200822214505611.png)

### 面试题

![image-20200822210355562](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200822210355562.png)

![image-20200822210340777](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200822210340777.png)

## 二、垃圾回收相关算法

### 标记阶段

#### 引用计数算法

引用计数算法（Reference Counting）：在对象中添加一个 `引用计数器` ，对于A对象，每当有一个对象引用了A，则A的引用计数器就加1；当引用失效时，引用计数器减1,。只要引用计数器为0就表示A对象可以被回收了。

优点：

- 实现简单，垃圾对象很容易识别；
- 判定效率高，回收没有延迟性。

 缺点：

- 致命缺点：很难解决对象之间 `循环引用` 的问题，这也==导致主流的Java虚拟机没有选用该算法管理内存==。

#### 可达性分析算法

当前主流的商用程序语言（Java、C#，上溯至前面提到的古老的Lisp）的内存管理子系统，都是通过可达性分析（Reachability Analysis）算法来判定对象是否存活的。

##### 算法思路

- 以 `GC Roots` 为起始节点集，从这些节点开始，根据引用关系向下搜索，搜索过程所走过的路径称为`引用链（ReferenceChain）`
- 如果某个对象到GC Roots间没有任何引用链相连，或者用图论的话来说就是从GC Roots到这个对象不可达时，则证明此对象是不可能再被使用的。

![image-20200823134917602](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200823134917602.png)

##### GC Roots

Java语言中，固定可作为 `GC Roots` 的对象包括以下几类元素：

- 虚拟机栈中引用的对象
  - 比如：各个线程被调用的方法堆栈中使用到的参数、局部变量等

- 本地方法栈内JNI（通常说的本地方法）引用的对象
- 方法区中类静态属性引用的对象
  - 比如：Java类的引用类型静态变量
- 方法区中常量引用的对象
  - 比如：字符串常量池（StringTable）里的引用
- 所有被同步锁（synchronized关键字）持有的对象
- Java虚拟机内部的引用
  - 比如：基本数据类型对应的Class对象，一些常驻的异常对象（比如NullPointExcepiton、OutOfMemoryError）等，还有系统类加载器
- 反映Java虚拟机内部情况的JMXBean、JVMTI中注册的回调、本地代码缓存等



> 小技巧：
> 由于Root采用栈方式存放变量和指针，所以如果一个指针，它保存了堆内存里面的对象，但是自己又不存放在堆内存里面，那它就是一个Root。

##### 注意

- 如果要使用可达性分析算法来判断内存是否可回收，那么 `根节点枚举工作` 必须在一个能保障一致性的快照中进行，不能出现分析过程中，根节点集合的对象引用关系还在发生变化。这点不满足的话分析结果的准确性就无法保证。

- 这点也是导致GC进行时必须 `Stop The World` 的一个重要原因。
  - 即使是号称(几乎)不会发生停顿的 `CMS收集器` 中，==枚举根节点时也是必须要停顿的==。	

#### 对象生存或死亡

##### 对象的finalization机制

如果从所有的 `GC Roots` 都无法访问到某个对象，说明对象已经不再使用了。一般来说，此对象需要被回收。但事实上，也并非是“非死不可”的，这时候它们暂时处于“缓刑”阶段。一个==无法触及的对象有可能在某一个条件下“复活”自己==，为此，定义虚拟机中的对象可能的三种状态。如下:

- `可触及的`：从根节点开始，可以到达这个对象。
- `可复活的`：对象的所有引用都被释放，但是对象有可能在finalize()中复活。
- `不可触及的`：对象的finalize()被调用，并且没有复活，那么就会进入不可触及状态。不可 触及的对象不可能被复活，因为finalize()只会被调用一次。

以上3种状态中，是由于 `finalize()` 方法的存在才进行的区分。只有在对象不可触及时才可以被回收。

##### 具体过程

![image-20200823144023394](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200823144023394.png)

> 虚拟机创建的 `Finalizer`线程会去触发 `finalize()`方法，但并不承诺一定会等待它运行结束。这样做的原因是，如果某个对象的finalize()方法执行缓慢，或者更极端地发生了死循环，将很可能导致F-Queue队列中的其他对象永久处于等待，甚至导致整个内存回收子系统的崩溃。

### 清除阶段

当成功区分出内存中存活对象和死亡对象后，GC 接下来的任务就是执行垃圾回收，释放掉无用对象所占用的内存空间，以便有足够的可用内存空间为新对象分配内存。

目前在JVM中比较常见的三种垃圾收集算法是标记-清除算法( `Mark - Sweep`)、复制算法(`Copying`)、标记一压缩算法(`Mark - Compact` )。

#### 标记-清除算法

 ##### 执行过程

当堆中的有效内存空间(`available memory`) 被耗尽的时候，就会停止整个程序(`stop the world`)，然后进行两项工作，第一项则是标记，第二项则是清除。

- ==标记==：Collector从引用根节点开始遍历，标记所有被引用的对象。一般是在对象的Header中记录为可达对象。
- ==清除==：Collector对堆内存从头到尾进行线性的遍历，如果发现某个对象在其Header中没有标记为可达对象，则将其回收。

##### 缺点

- 执行效率不高，如果大部分的对象需要回收，则需要大量的标记和清除动作。
- 内存空间的碎片化问题。需要维护一个空闲列表。

![image-20200823220111548](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200823220111548.png)

#### 复制算法

 ##### 核心思想

u = 使用中的内存块

n = 未使用的内存块

把内存空间分成两块，每次只使用其中的一块，在垃圾回收时将`u`中的对象复制到 `n`，之后清除`u`中的所有对象，交换两个内存块的角色，完成垃圾回收。

![image-20200824092210116](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200824092210116.png)

##### 优缺点

优点：

- 实现简单，运行高效
- 不会出现“碎片”问题。

缺点：

- 需要两倍的内存空间。
- 对于G1这种分拆成为大量region的GC，复制而不是移动，意味着GC需要维护region之间对象引用关系，不管是内存占用或者时间开销也不小。

特别的：

- 适合==可回收对象多==的情况。

> 因为新生代中的对象具有 `朝生夕死` 的特点，因此并不需要按照 1:1 的比例来划分新生代的内存空间。使用的是“Apple式回收”：把新生代划分为较大的`Eden空间`和两块较小的`Survivor空间`。

#### 标记-整理算法

又称`标记 - 压缩`算法，主要是针对老年代的存活特征。

##### 执行过程

1. 标记过程和 `标记 - 清除` 算法一样；

2. 让所有存活的对象都向内存空间一端移动；
3. 清除边界外所有的空间。

![image-20200824101521208](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200824101521208.png)

##### 优缺点

![image-20200824103047947](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200824103047947.png)

#### 总结

![image-20200824102959140](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200824102959140.png)

### 分代收集算法

![image-20200824104545629](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200824104545629.png)

### 增量收集算法

为了避免在收集垃圾时的长时间的停顿（因为 `Stop The World`），可以允许垃圾收集器以==分阶段的方式完成标记、清理或复制工作==。

#### 缺点

使用这种方式，由于在垃圾回收过程中，间断性地还执行了应用程序代码，所以能减少系统的停顿时间。但是，因为==线程切换和上下文转换的消耗，会使得垃圾回收的总体成本上升，造成系统吞吐量的下降==。



## 三、垃圾回收相关概念

### `System.gc()` 的理解

在默认情况下，通过`System.gc ()`或者`Runtime.getRuntime().gc()` 的调用，会显式触发`Full GC`，同时对老年代和新生代进行回收，尝试释放被丢弃对象占用的内存。

然而 `System.gc()` 调用附带一个免责声明，无法保证对垃圾收集器的调用。

### 内存溢出与内存泄漏

#### 内存溢出(OOM)

`OutOfMemory`: ==没有空闲内存，并且垃圾收集器也无法提供更多的内存==。

在抛出 `OutOfMemoryError` 之前，通常垃圾收集器都会被触发，尽可能清理出空间。

- 例如：在引用机制分析中，涉及到JVM会去尝试回收 `软引用指向的对象等`。
- 在 `java.nio.BIts.reserveMemory()` 方法中，可以看到 `System.gc() `被调用。

> 但并不是任何情况下都会触发垃圾收集。
>
> - 比如，分配的对象超大，超过了堆的最大值。就会直接报 `OutOfMemoryError`

#### 内存泄漏（Memory Leak）

也称作`存储渗漏`。==严格来说：只有对象不会再被使用，但是 GC 又不能回收==，才叫做内存泄漏。

但实际情况下，一些不太好的实践(或疏忽)会导致对象的生命周期变得很长甚至导致00M，也可以叫做==宽泛意义上的“内存泄漏”==。  

内存泄漏会蚕食可用内存，直至耗尽所有内存，导致出现   `OutOfMemoryError` 异常。

![image-20200824213556929](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200824213556929.png)

### Stop The World

GC 事件发生过程中，会产生应用程序的停顿。停顿产生时，整个应用程序都会被暂停，没有任何响应。

- 可达性分析算法中`枚举根节点(GC Roots)`会导致所有Java执行线程停顿。
  - 分析工作必须在一个能确保一致性的快照中进行；
  - `一致性`指整个分析期间整个执行系统看起来像被冻结在某个时间点上；
  - 如果出现分析过程中对象引用关系还在不断变化，则分析结果的准确性无法保证。

需要减少STW的发生。

所有的GC都会有 STW 事件，只能说垃圾回收器越老越优秀，回收效率越来越高，尽可能的缩短了暂停时间。

### 安全点与安全区域

#### 安全点

程序执行时并非在所有地方都能停顿下来开始GC，只有在特定的位置才能停顿下来开始GC，这些位置称为`安全点（Safepoint）`。
`Safe Point`的选择很重要，==如果太少可能导致Gc等待的时间太长，如果太频繁可能导致运行时的性能问题==。大部分指令的执行时间都非常短暂，通常会根据“==是否具有让程序长时间执行的特征==”为标准。比如:选择一些执行时间较长的指令作为Safe Point，如`方法调用`、`循环跳转`和`异常跳转`等。

> ![image-20200825180120500](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200825180120500.png)

#### 安全区域

`Safepoint` 无法解决程序不执行的情况，例如线程处于 `Sleep 状态或 Blocked 状态`，这时候就需要安全区域`Safe Region` 来解决。

安全区域是指在一段代码片段中，引用关系不会发生变化，在该区域的任何位置都可以 GC。

实际执行时：

1. 当线程运行到 `safe Region` 的代码时，首先标识已经进入了 `Safe Region`，如果这段时间内发生Gc，JVM会忽略标识为Safe Region状态的线程;
2. 当线程即将离开`safe Region`时，会检查JVM是否已经完成GC，如果完成了，则继续运行，否则线程必须等待直到收到可以安全离开`safe Region`的信号为止;

### 强引用

在代码中普遍存在的引用赋值，==任何情况下，垃圾收集器都不会回收还在被强引用的对象==。

### 软引用

在即将要发生OOM时，才会回收这些对象。如果回收后内存还是不够，才会抛出  `OOM`。

### 弱引用

被弱引用关联的对象==只能生存到下一次垃圾收集之前==。当垃圾收集器工作时，无论内存空间是否足够，都会回收掉被弱引用关联的对象。

> `WeakHashMap` 主要用来实现缓存，通过使用 `WeakHashMap` 来引用缓存对象，由 JVM 对这部分缓存进行回收。
>
> Tomcat 中的 `ConcurrentCache` 使用了` WeakHashMap` 来实现缓存功能。
>
> `ConcurrentCache` 采取的是分代缓存：
>
> - 经常使用的对象放入 eden 中，eden 使用 ConcurrentHashMap 实现，不用担心会被回收（伊甸园）；
> - 不常用的对象放入 longterm，longterm 使用 WeakHashMap 实现，这些老对象会被垃圾收集器回收。
> - 当调用 get() 方法时，会先从 eden 区获取，如果没有找到的话再到 longterm 获取，当从 longterm 获取到就把对象放入 eden 中，从而保证经常被访问的节点不容易被回收。
> - 当调用 put() 方法时，如果 eden 的大小超过了 size，那么就将 eden 中的所有对象都放入 longterm 中，利用虚拟机回收掉一部分不经常使用的对象。
>
> ```java
> public final class ConcurrentCache<K, V> {
> 
>     private final int size;
> 
>     private final Map<K, V> eden;
> 
>     private final Map<K, V> longterm;
> 
>     public ConcurrentCache(int size) {
>         this.size = size;
>         this.eden = new ConcurrentHashMap<>(size);
>         this.longterm = new WeakHashMap<>(size);
>     }
> 
>     public V get(K k) {
>         V v = this.eden.get(k);
>         if (v == null) {
>             v = this.longterm.get(k);
>             if (v != null)
>                 this.eden.put(k, v);
>         }
>         return v;
>     }
> 
>     public void put(K k, V v) {
>         if (this.eden.size() >= size) {
>             this.longterm.putAll(this.eden);
>             this.eden.clear();
>         }
>         this.eden.put(k, v);
>     }
> }
> ```

### 虚引用

一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来获得一个对象的实例。为一个对象设置虚引用关联的==唯一目的就是能在这个对象被收集器回收时收到一个系统通知==。

> 强引用（StrongReferenc）、软引用（Soft Reference）、弱引用（Weak Reference）、虚引用（Phantom Reference），这`4中引用强度依次减弱`。

## 四、垃圾回收器

### 评估GC的性能指标

- ==**吞吐量：运行用户代码的时间占总运行时间的比例。**==
- ==**暂停时间：执行垃圾收集时，程序的工作线程被暂停时间。**==
- ==内存占用：Java 堆区所占用的内存大小。==
- 收集频率：相对于应用程序的执行，收集操作发生的频率。

### 不同的垃圾回收器概述

![image-20200826230138175](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200826230138175.png)

#### 7种经典收集器与垃圾分代之间的关系

![image-20200827090432083](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200827090432083.png)

#### 垃圾收集器的组合关系

![image-20200827092414610](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200827092414610.png)

> 1. 两个收集器之间有连线，表明它们可以搭配使用。
> 2. 其中 `Serial Old GC` 作为 `CMS GC` 出现 `Concurrent Mode Failure` 失败后的后备预案。
> 3. （红色虚线）由于维护和兼容性测试的成本，在JDK 8时将`Serial+CMS`、`ParNew+Serial Old`这两个组合声明为废弃（EP 173），并在JDK 9中完全取消了这些组合的支持（EP214），即：移除。
> 4. （绿色虚线）JDK14中，弃用`Parallel Scavenge` 和 `Serial Old GC` 组合。
> 5. （青色虚线）JDK14中，删除了CMS垃圾回收器。

#### 如何查看默认的垃圾收集器

![image-20200827093505384](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200827093505384.png)

### Serial回收器：串行回收

这个收集器是一个 `单线程工作` 的收集器，但它的“单线程”并不只是说它==只会使用一个CPU或一条收集线程去完成垃圾收集工作==，更重要的是在它进行垃圾收集时，==必须暂停其它所有工作的线程==，直到它收集结束。

![image-20200827095954119](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200827095954119.png)

Serial回收器的优点是：

- 简单而高效（与其它收集器的单线程相比），对于内存资源受限的环境，它是所有收集器里==额外内存消耗最小==的。对于单核CPU来说，Serial由于没有线程交互的开销，因为有着更高的垃圾收集效率。
- 在用户的桌面应用场景中，可用内存一般不大（几十MB至一两百MB），可以在较短时间内完成垃圾收集（几十ms至一百多ms），只要不频繁发生，使用串行回收器是可以接受的。
- 在HotSpot虚拟机中，使用`–XX:+UseSerialGc`参数可以指定年轻代和老年代都使用串行收集器。等价于新生代用`serial Gc`，且老年代用`serial old Gc`。

### ParNew回收器：并行回收

`ParNew` 收集器实质上是 `Serial` 收集器的多线程并行版本，并没有多大的创新之处。

到现在官方更希望它完全被 `G1` 所取代，加上取消了`Serial+CMS`、`ParNew+Serial Old`的组合使用，到目前 `ParNew` 只能和 `CMS` 搭配使用。

![image-20200827102704674](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200827102704674.png)

> 在单核心的环境下，ParNew收集器绝对不会有比Serial收集器更好的效果。 

### Parallel回收器：吞吐量优先

`Parallel Scavenge` 收集器与其他收集器的不同在于，其它收集器的关注点是尽可能减少 `Stop The World` 的时间，而 `Parallel Scavenge` 更关注达到一个可控制的吞吐量（Throughput）。

> 吞吐量就是处理器用于运行用户代码的时间与处理器总消耗时间的比值。
>
> 停顿时间越短就越适合与用户交互的程序，良好的响应速度能提升用户体验。
>
> 高吞吐量则可以最高效率利用处理器资源，尽快完成运算任务，主要适合后台运算的分析任务。例如工资支付、科学计算、日志分析的应用。

`自适应调节策略` 是 `Parallel Scavenge` 与 `ParNew` 的一个重要区别。`自适应调节策略` 是指虚拟机会根据当前系统的运行情况，动态调整虚拟机参数以提供最合适的停顿时间或者最大的吞吐量。

![image-20200827105448084](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200827105448084.png)

> 在Java8中，默认使用此垃圾收集器。

### CMS回收器：低延迟

`Concurrent-Mark-Sweep GC` 。 

`CMS` 是一个==以获取最短回收停顿时间为目标==的收集器。这款收集器是 `HotSpot` 虚拟机第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程（基本上）同时工作。

> 目前很大一部分的Java应用集中在互联网站或者B/S系统的服务端上，这类应用尤其重视服务的响应速度，希望系统停顿时间最短，以给用户带来较好的体验。
> `CMS收集器` 就非常符合这类应用的需求。

CMS收集器是采用` 标记-清除` 算法。

![image-20200827112747789](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200827112747789.png)

1. 初始标记（`initial mark`）：仅仅只是标记一下 `GC Roots` 能直接关联到的对象，速度很快；
2. 并发标记（`concurrent mark`）：从`GC Roots`的直接关联对象开始遍历整个对象图的过程，这个过程比较耗时，但不需要 `STW`。
3. 重新标记（`remark`）：修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录。这个阶段的停顿时间通常会比`初始标记阶段`稍长一些，但也远比并发标记阶段的时间短。
4. 并发清除（`concurrent sweep`）：清理删除掉标记阶段判断已经死亡的对象，==由于不需要移动存活对象，所以这个阶段也不需要 `STW`==，当然这也导致了只能使用`Mark-Sweep` 算法，会造成内存碎片问题，进而不能使用`指针碰撞`方法分配对象所需内存空间（使用 `空闲列表` 法）。

#### 优缺点

优点：

- 并发收集
- 低延迟

缺点：

- 使用“标记-清除”算法，会造成内存碎片。
- CMS收集器对CPU资源非常敏感。在并发阶段，它虽然不会导致用户停顿，但是会因为占用了一部分线程而导致应用程序变慢，总吞吐量会降低。
- CMS收集器无法处理`浮动垃圾`。可能出现`Concurrent Mode Failure`失败而导致另一次`Full GC`的产生。在并发标记和并发清理阶段，用户线程还在继续运行，那么就会产生新的垃圾对象，但因为这些垃圾出现在标记过程之后，CMS无法在此次收集中处理掉它们，只能留在下一次GC时清理。这一部分垃圾就是 “浮动垃圾”。

### G1回收器：区域化分代式

官方给G1设定的目标是==在延迟可控的情况下获得尽可能高的吞吐量==，所以才担当起了`全功能收集器` 的重任与期望。

G1是一款面向服务端应用的垃圾收集器，==主要针对配备多核CPU及大容量内存的机器==。是JDK9以后的默认垃圾收集器。

#### G1的优势与不足

优点：

- 并行与并发：
  - 并行性：G1在回收期间，可以有多个GC线程同时工作，有效利用多核计算能力。此时用户线程STW。
  - 并发性：G1拥有与应用程序交替执行的能力，部分工作可以和应用程序同时执行，因此，一般来说，不会在整个回收阶段发生完全阻塞应用程序的情况。
- 分代收集
  - 从分代上看，G1依然属于分代型垃圾回收器，它会区分年轻代和老年代，年轻代依然有Eden区和survivor区，但从堆的结构上看，它不要求整个Eden区、年轻代或者老年代都是连续的，也不再坚持固定大小和固定数量。
  - 将堆空间划分为多个大小相等的独立区域（`Region`），每一个`Region`都可以根据需要，扮演新生代的Eden空间、Survivor空间，或者老年代空间。收集器能够==对扮演不同角色的`Region`采用不同的策略去处理==。
- 空间整合
  - G1将内存划分为一个个的region。内存的回收是以region作为基本单位的。
    ==Region之间是标记-复制算法==，但整体上可看作是==标记-压缩(Mark-Compact)算法==，两种算法都可以避免内存碎片。这种特性有利于程序长时间运行，分配大对象时不会因为无法找到连续内存空间而提前触发下一次GC。尤其是当Java堆非常大的时候，G1的优势更加明显。
- 可预测的停顿时间模型
  - 这是G1相对于CMS 的另一大优势，G1除了追求低停顿外，还能建立`可预测的停顿时间模型`，能让使用者==明确指定在一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒==。
  - G1的具体回收思路是：G1收集器去跟踪各个Region里面的垃圾堆积的价值大小（价值即回收所获得的空间大小以及回收所需时间的经验值），然后在后台维护一个优先级列表，==每次根据允许的收集时间，优先回收价值最大的Region==。保证了G1收集器在有限的时间内获取尽可能高的收集效率。

缺点：

- 相较于CMS，G1还不具备全方位、压倒性优势。比如在用户程序运行过程中，G1无论是为了垃圾收集产生的==内存占用(Footprint）==还是程序运行时的==额外执行负载（overload）==都要比CMS要高。
- 从经验上来说，在小内存应用上CMS的表现大概率会优于G1，而==G1在大内存应用上则发挥其优势==。平衡点在6-8GB之间。

#### G1回收器的适应场景

- 面向服务端应用，针对具有大内存、多处理器的机器。(在普通大小的堆里表现并不惊喜)
- 最主要的应用是需要低GC延迟，并具有大堆的应用程序提供解决方案；
- 如：在堆大小约6GB或更大时，可预测的暂停时间可以低于0.5秒；(G1通过每次只清理一部分而不是全部的Region的增量式清理来保证每次GC停顿时间不会过长）。
- 用来替换掉JDK1.5中的CMS收集器；在下面的情况时，使用G1可能比CMS好：
  - 超过50%的Java堆被活动数据占用；
  - 对象分配频率或年代提升频率变化很大；
  - GC停顿时间过长（长于0.5至1秒）
- HotSpot垃圾收集器里，除了G1以外，其他的垃圾收集器使用内置的JVM线程执行GC的多线程操作，而G1 GC==可以采用应用线程承担后台运行的GC工作==，即当JVM的GC线程处理速度慢时，系统会调用应用程序线程帮助加速垃圾回收过程。

#### Region：化整为零

![image-20200827174341535](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200827174341535.png)

G1垃圾收集器中还有一类特殊的`Humongous`区域，==专门用来存储大对象==。G1认为只要大小超过了一个Region容量一半的对象即可判定为大对象。

如果一个Region装不下一个大对象，那么该对象会被放在N个==连续的H区==中。

> Region区也使用了`TLAB`来保证多线程下对象内存空间的快速分配。
>
> 为什么要有H区？
>
> 对于堆中的大对象，默认直接会被分配到老年代，但是如果它是一个短期存在的大对象，就会对垃圾收集器造成负面影响。为了解决这个问题，G1划分了一个Humongous区。

#### Remembered Set

- 一个对象会被不同区域所引用。
- 垃圾收集器在回收新生代的垃圾对象时，为了避免把整个老年代加入 `GC Roots` 扫描范围，建立了 记忆集（`Remembered Set`）。
- 实际上，所有的`部分区域收集`行为的垃圾收集器，都会面临相同的问题。

解决方法：

- 无论G1还是其他分代收集器，JVM都是使用`Remembered Set`来避免全局扫描：
- 每个`Region`都有一个对应的`Remembered set`。
- 每次`Reference`类型数据写操作时，都会产生一个`写屏障（write Barrier）`暂时中断操作。
- 然后检查指向对象是否和`Reference`在不同的`Region`(对于其他收集器：检查老年代对象是否引用了新生代对象）。
- 如果不同，通过`卡表（CardTable）`把相关引用信息记录到指向对象的所在Region对应的`Remembered set`中；
- 当进行垃圾收集时，在GC根节点的枚举范围加入`Remembered Set`，就可以保证不进行全局扫描，也不会有遗漏。

![image-20200827200611235](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200827200611235.png)

#### G1回收器垃圾回收过程

![image-20200827202645919](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200827202645919.png)

- 初始标记（`Initial Marking`）：仅仅==只是标记一下GC Roots能直接关联到的对象==，并且修改TAMS指针的值，让下一阶段用户线程并发运行时，能正确地在可用的Region中分配新对象。这个阶段需要停顿线程，但耗时很短，而且是借用进行Minor GC的时候同步完成的，所以G1收集器在这个阶段实际并没有额外的停顿。
- 并发标记（`Concurrent Marking`）：从GC Root开始对堆中对象进行可达性分析，==递归扫描整个堆里的对象图，找出要回收的对象==，这阶段耗时较长，但可与用户程序并发执行。当对象图扫描完成以后，还要重新处理SATB记录下的在并发时有引用变动的对象。
- 最终标记（`Final Marking`）：对用户线程做另一个短暂的暂停，用于==处理并发阶段结束后仍遗留下来的最后那少量的SATB记录==。
- 筛选回收（`Live Data Counting and Evacuation`）：负责==更新Region的统计数据，对各个Region的回收价值和成本进行排序==，根据用户所期望的停顿时间来==制定回收计划==，可以自由选择任意多个Region构成回收集，然后把决定回收的那一部分Region的存活对象复制到空的Region中，再清理掉整个旧Region的全部空间。这里的操作涉及存活对象的移动，是必须暂停用户线程（STW），由多条收集器线程并行完成的。

![image-20200827205412079](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200827205412079.png)

### 总结

![image-20200827205523221](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200827205523221.png)
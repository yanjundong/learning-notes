# JVM的压缩指针

为了用32位oop（ordinary object pointer）来引用更大的堆内存，需要使用压缩指针。

实现方法是不再保存所有引用，而是每隔8个字节保存一个引用。例如，原来保存每个引用用0、1、2...，现在只保存0、8、16...。因此，指针压缩后，并不是所有引用都保存在堆中，而是以8个字节为间隔保存引用。

在实现上，堆中的引用其实还是按照0x0、0x1、0x2...进行存储。只不过当引用被存入64位的寄存器时，JVM将其左移3位（相当于末尾添加3个0），例如0x0、0x1、0x2...分别被转换为0x0、0x8、0x10。而当从寄存器读出时，JVM又可以右移3位，丢弃末尾的0。（oop在堆中是32位，在寄存器中是35位，2的35次方=32G。也就是说，==使用32位，来达到35位oop所能引用的堆内存空间==）。

在JVM中（不管是32位还是64位），对象已经按8字节边界对齐了。对于大部分处理器，这种对齐方案都是最优的。所以，使用压缩的oop并不会带来什么损失，反而提升了性能。

> Oracle JDK从6 update 23开始在64位系统上会默认开启压缩指针。
>
> 32位HotSpot VM是不支持UseCompressedOops参数的，只有64位HotSpot VM才支持。
>
> 对于大小在4G和32G之间的堆，应该使用压缩的oop。

[压缩指针参考](https://juejin.cn/post/6844903768077647880)

## finalize方法中抛出异常会怎么样？

当`finalize()`抛出异常的时候会被忽略。而且，对象的终结将在此停止（也就是finalize方法在抛出异常的地方被视作finalize方法执行完成）， 之后**对象对应的堆内存处于可回收状态。**

# 根节点枚举、安全点、安全区域

为了能够在GC时能够快速找到 GC Roots，而不用每次进行GC时都去所有的栈和整个方法区查找对象引用，因此引用了`OopMap(Ordinary Object Point)`的概念。

虚拟机使用这个数据结构，可以直接知道哪些地方存放着对象引用。这样，就可以在GC扫描时快速知道这些信息，然后进行可达性分析。

导致`OopMap`变化的指令很多，如果为每条指令都生成对应的`OopMap`，那就需要很大的存储空间，是不可能实现的。因此，虚拟机选择在 “特定的位置” 记录了这些信息，这些位置就是 `安全点`。对于安全点的选择，既不能太少以至于让收集器等待时间太长，也不能太多以至于过分增大运行时的内存负荷。

==安全点的选定==基本上是以程序“是否具有让程序长时间执行的特征”为标准进行选定的，因为每条指令执行的时间都非常短暂，程序不太可能因为指令流长度太长这个原因而过长时间运行，“长时间运行”的最明显特征就是指令序列复用，例如方法调用、循环跳转、异常跳转等，所以具备这些功能的指令才会产生SafePoint。 

**再总结一下，安全点的位置主要在：** 

- 循环的末尾
- 方法临返回前/调用方法的call指令后
- 可能抛异常的位置

有些程序并没有执行，没有分配处理器时间，因此也就不会运行到安全点去检查标志位是否为true，以判断是否要将自己挂起，例如，用户线程处于 sleep或者blocked状态。这时就需要引入安全区域，在该区域中，不会导致引用关系的改变。

# 什么时候触发GC？

-XX:CMSInitiatingOccupancyFraction=70 可以设置当老年代的空间占用达到70%时就会触发老年代的GC。

-XX：ParallelGCThreads可以限制ParNew的GC线程数 

**问题：**如果一次CMS GC之后，老年代的空间占用比仍然大于CMSInitiatingOccupancyFraction指定的阈值，会不会继续进行CMS GC呢？

# 垃圾收集器

## G1回收器：区域化分代式

### G1的Remembered Set

每个Region都维护有自己的 Remembered Set，G1的记忆集在结构上是一个哈希表，key是别的Region的起始地址，value是一个集合，存储的元素是卡表的索引号，这种“双向”的卡表结构，还记录了“谁指向我”。因此实现起来更加复杂。

### G1在并发标记阶段如何保证收集线程和用户线程互不干扰地运行？

- 首先需要解决用户线程改变对象引用关系时，必须保证不能打破原本的对象图结构，导致标记结果出现错误。针对这个问题：CMS采用增量更新算法实现，而G1采用原始快照(`SATB`)算法实现。
- 在垃圾回收时，新创建对象的内存分配上（类似于CMS）。G1为每个Region设计了两个名为TAMS的指针，并发回收时新分配的地址都必须在这两个指针位置以上。G1在回收时，默认该区域内的对象是存活的，不会回收。如果内存回收的速度赶不上新对象分配的速度，也会出现并发失败，导致Full GC产生长时间的STW。

# Java内存泄漏实例

比如将一个对象作为key值存入一个HashMap中，之后改变了这个对象的hashCode值，那么之前存入到HashMap中的这个KV就找不到了，这个对象及Value就一直不能被GC看，最终可能导致OOM。

# OOM异常相关

在Java运行时数据区中，除了程序计数器外，其余部分都会发生OOM异常。

## Java堆溢出

只要我们不断地创建对象，并且保证GC Roots 到对象之间有可达路径来避免被GC，随着对象数量的增加，在总容量达到堆的限制后就会发生OOM。

```java
/** 通过 -Xms20M -Xmx20M 限制Java堆的大小为20MB */
public class HeapOOM {

    private static List<String> oomList = new ArrayList<>();
    private static int count = 0;

    public static void main(String[] args) {

        while (true) {
            oomList.add("AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA");
            System.out.println(++count);
        }

    }
    /*
     * Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
     * 	at java.util.Arrays.copyOf(Arrays.java:3210)
     * 	at java.util.Arrays.copyOf(Arrays.java:3181)
     * 	at java.util.ArrayList.grow(ArrayList.java:265)
     * 	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:239)
     * 	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:231)
     * 	at java.util.ArrayList.add(ArrayList.java:462)
     * 	at com.coder.learnjava.HeapOOM.main(C.java:19)
     */

}
```

## 虚拟机栈和本地方法栈溢出

- 线程请求的深度超出虚拟机所允许的最大深度，会抛出 `StackOverflowError` 异常。

- 如果虚拟机在扩展栈时无法申请到足够的内存空间，则抛出OOM异常。

  - 如果线程请求的栈容量大于虚拟机所允许的最大容量，将抛出 `StackOverflowError` 异常；如果Java虚拟机栈容量可以动态扩展，当栈扩展时无法申请到足够的内存会抛出 `OutOfMemoryError`异常。

  - > HotSpot虚拟机的栈容量是不可以动态扩展的，所以HotSpot虚拟机的虚拟机栈不存在 `OOM` 。但是如果在申请时就失败，仍会OOM异常。

## 方法区溢出

不断使用 ASM 工具动态生成Class并使用JVM加载，并在list中持有这些对象不释放导致不能类卸载，知道超出方法区大小。

## 本地直接内存溢出

不断使用UnSafe.allocateMemory分配堆外内存就会出现直接内存溢出的情况。

## 排查内存溢出的工具

排查jvm问题最有效的命令就是jvm自带的jmap、jstat、jstack等，他们分别用来定位内存问题和堆栈问题，如果对线上有权限，JVisualVM也可以使用。

**jvm相关有效命令的一个列表：**

![enter image description here](images/clip_image002.jpg)

### Jstat使用方法：

![img](images/clip_image004.jpg)

![img](images/clip_image006.jpg)

![img](images/clip_image007.png)

![img](images/clip_image009.jpg)

### Jmap使用方法：

![img](images/clip_image011.jpg)

使用jmap还可以导出内存映像文件。Jmap –dump xxx

# Java多线程死锁的检测与预防

死锁，是指多个进程因争夺资源而造成的一种僵局，当进程处于这种状态时，若无外力外用，它们都将无法向前推进。举个例子，如果线程1获得了资源A，等待获取资源B，但线程2获得了资源B，等待获取资源A。

![image-20201228155651096](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201228155651096.png)

用一段代码来模拟：

```java
public static void main(String[] args) {
    final Object a = new Object();
    final Object b = new Object();
    Thread threadA = new Thread(new Runnable() {
        public void run() {
            synchronized (a) {
                try {
                    System.out.println("now i in threadA-locka");
                    Thread.sleep(1000l);
                    synchronized (b) {
                        System.out.println("now i in threadA-lockb");
                    }
                } catch (Exception e) {
                    // ignore
                }
            }
        }
    });

    Thread threadB = new Thread(new Runnable() {
        public void run() {
            synchronized (b) {
                try {
                    System.out.println("now i in threadB-lockb");
                    Thread.sleep(1000l);
                    synchronized (a) {
                        System.out.println("now i in threadB-locka");
                    }
                } catch (Exception e) {
                    // ignore
                }
            }
        }
    });

    threadA.start();
    threadB.start();
}

```

## Jstack 命令

![image-20201228155856971](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201228155856971.png)

## jConsole命令

Jconsole是JDK自带的监控工具，在JDK/bin目录下可以找到。它用于连接正在运行的本地或者远程的JVM，对运行在Java应用程序的资源消耗和性能进行监控，并画出大量的图表，提供强大的可视化界面。而且本身占用的服务器内存很小，甚至可以说几乎不消耗。

我们在命令行中敲入jconsole命令，会自动弹出以下对话框，选择进程1362，并点击“**链接**”

![image-20201228160425857](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201228160425857.png)

![image-20201228160500093](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201228160500093.png)

![image-20201228160611570](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201228160611570.png)

可以看到进程中存在死锁。以上例子我都是用synchronized关键词实现的死锁，如果读者用ReentrantLock制造一次死锁，再次使用死锁检测工具，也同样能检测到死锁，不过显示的信息将会更加丰富。

## 死锁发生的条件

产生死锁的必要条件：

1. 互斥条件：进程要求对所分配的资源进行排它性控制，即在一段时间内某资源仅为一进程所占用。
2. 请求和保持条件：当进程因请求资源而阻塞时，对已获得的资源保持不放。
3. 不剥夺条件：进程已获得的资源在未使用完之前，不能剥夺，只能在使用完时由自己释放。
4. 环路等待条件：在发生死锁时，必然存在一个进程--资源的环形链。

## 预防死锁

1. 以确定的顺序获得锁
2. 超时放弃

## 避免死锁

系统在进行资源分配之前预先计算资源分配的安全性。若此次分配不会导致系统进入不安全的状态，则将资源分配给进程；否则，进程等待。其中最具有代表性的避免死锁算法是银行家算法。

[Java面试必问-死锁终极篇](https://juejin.cn/post/6844903577660424206)

# 系统CPU比较高的原因

1. 程序中存在类似于while true这样一直占用cpu的死循环，==注意阻塞在synchronized上的死锁线程并不会很占用cpu，因为这种死锁线程都处于阻塞状态==。
2. GC很频繁，GC线程属于CPU密集型，GC过程一般是并发执行的。

如何定位最消耗CPU的线程：

Top（找到是哪个线程最占用cpu） + jstack（首先通过jps确定我们当前执行java的任务的进程号，之后找到此线程对应的java进程）

参考：[如何定位消耗CPU最多的线程](http://lovestblog.cn/blog/2016/03/31/cpu-thread/)

[top+jstack分析cpu过高原因](https://blog.csdn.net/z69183787/article/details/81392213)

# 类加载方面

## 自定义类加载器

### 经典实现：Tomcat

<img src="https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201228113501676.png" alt="image-20201228113501676" style="zoom:80%;" />

**Tomcat**自定义类加载器的目的是为了支持对目录里的类库进行加载和隔离。比如CommonClassLoader加载的类都可以被CatalinaClassLoader和SharedClassLoader使用，而CatalinaClassLoader与SharedClassLoader自己加载的类则与对方相互隔离。

### ClassLoader.loadClass和Class.forName异同：

1. ClassLoader.loadClass得到的class是还没有链接的(即验证、准备、解析阶段都还没有进行，类初始化阶段自然也没有做)，Class.forName得到的class是已经初始化完成的。也就是说使用ClassLoader.loadClass加载类，类中的静态成员变量不会被初始化，类中的静态代码块也不会被执行；使用Class.forName则会被执行。

2. Class.forName加载类使用的类加载器是与调用forName的对象所在类的类加载器一致的；ClassLoader.loadClass则是使用我们指定的类加载器进行加载。

## 为什么类加载器不实现主动卸载类的功能？

因为ClassLoader是负责加载类的，并不会去关心加载的类会被谁使用，既然不知道加载的类会被谁使用，也就不知道类是不是没有任何其他引用了。

# 创建和销毁对象

## 使用静态工厂方法创建对象

除了公有的构造器外，类还可以提供一个公有的`静态工厂方法`来返回类的实例。例如：

```java
public static Integer valueOf(String s) throws NumberFormatException {
    return Integer.valueOf(parseInt(s, 10));
}
```

提供静态工厂方法而不是公有的构造方法，这样做的优缺点：

**优点：**

- 静态工厂方法有名称，能够更清楚的表明这个方法的作用；

- 静态工厂方法不必在每次调用的时候都创建一个新的对象，例如 `Integer` 的 `IntegerCache` ；

- 静态工厂方法可以返回原返回类型的任何子类型的对象，这样的灵活性在于可以返回类的对象，但又不需要把类变成公有的，例如：

  ```java
  public class Collections {
      // 类为私有的
      private static class SynchronizedMap<K,V>
          implements Map<K,V>, Serializable {
          private static final long serialVersionUID = 1978198479659022715L;
  
          private final Map<K,V> m;     // Backing Map
          final Object      mutex;        // Object on which to synchronize
  
          SynchronizedMap(Map<K,V> m) {
              this.m = Objects.requireNonNull(m);
              mutex = this;
          }
          // ...
      }
      
      // 返回 Map 的子类对象
      public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m) {
          return new SynchronizedMap<>(m);
      }
  }
  ```

- 静态工厂方法所返回的对象的类可以随着每次调用而发生变化；

- 方法返回的对象所属的类，在编写包含该静态工厂方法的类时可以不存在；

**缺点：**

- 类如果不包含公有的或者受保护的构造器，就不能被子类化；

- 程序员很难发现它们，但可以通过规范命名习惯来避免该问题：

  ```java
  // from - 类型转换方法，只有单个参数，返回该类型的一个相对应的实例
  Date d = Date.from(instant);
  
  // of - 聚合方法，带有多个参数，返回该类型的一个实例
  LocalDate date = LocalDate.of(1997, 10, 1);
  
  // valueOf - 上述两种方法的一种替代方法
  Integer i = Integer.valueOf("123");
  
  // instance 或 getInstance - 返回的实例是通过参数来描述的
  StackWalker luke = StackWalker.getInstance(options);
  
  // create 或 newInstance - 保证每次调用都返回一个新的实例
  
  // getType - 像 getInstance 一样，但是在工厂方法处于不同的类中的时候使用
  FileStore fs = Files.getFileStore(path);
  
  // newType - 像 newInstance 一样，但是在工厂方法处于不同的类中的时候使用
  BufferedReader br = Files.newBufferedReader(path);
  
  // type - getType 和 newType 的简称
  List<Complanit> litany = Collections.list(legacyLitany);
  ```

## 遇到多个构造器参数时考虑构建器

静态工厂方法和构造器都有个缺点：它们都不能很好的扩展到大量的可选参数。

第一种方法可以通过构造方法的重载来解决，但是当参数过多时 就会失去控制；

第二种方法可以使用 `JavaBeans` 模式，通过多次调用 `setter` 方法来解决，但这样的话 在构造过程中 JavaBean 可能会处于不一致的状态；

第三种方法使用 `建造者(builder)模式` ；

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        /* 必要参数 */
        private final int servingSize;
        private final int servings;
        /* 可选参数 */
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }
        public Builder calories(int val) {
            this.calories = val;
            return this;
        }
        public Builder fat(int val) {
            this.fat = val;
            return this;
        }
        public Builder sodium(int val) {
            this.sodium = val;
            return this;
        }
        public Builder carbohydrate(int val) {
            this.carbohydrate = val;
            return this;
        }
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        this.servingSize = builder.servingSize;
        this.servings = builder.servings;
        this.calories = builder.calories;
        this.fat = builder.fat;
        this.sodium = builder.sodium;
        this.carbohydrate = builder.carbohydrate;
    }

    // 调用
    public static void main(String[] args) {
        NutritionFacts cocaCola = new Builder(240, 8).calories(100).sodium(35).build();
    }
}
```

## 用私有化构造器或枚举型强化Singleton

方法一：公有的静态final域

```java
public class Singleton {
    public static final Singleton INSTANCE = new Singleton();
    private Singleton() { }
    
    public void method() {
        // ...
    }
}
```

方法二：饿汉式

```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();
    private Singleton() { }

    public Singleton getInstance() {
        return INSTANCE;
    }
    
    public void method() {
        // ...
    }
}
```

方法三：声明一个包含单个元素的枚举类型，该方法与方法一类似，但更加简洁，而且提供了序列化机制：

```java
public enum Singleton {
    INSTANCE;

    public void method() {
        // ...
    }
}
```

> 如果 `Singleton` 必须扩展一个超类，而不是扩展 Enum 时，则不该使用这个方法。

## 通过私有构造器强化不可实例化的能力

有一些工具类，比如`Collections`等，它们仅仅是提供一些能力，自己本身不具备任何属性，所以，不希望被实例化。然而，缺失构造函数时编译器会自动添加上一个无参的构造器。所以，需要提供一个私有化的构造函数。为了防止在类内部误用，再加上一个保护措施和注释。

```java
public class UtilityClass {
    // 禁止使用默认构造函数，以确保不会被实例化。
    private UtilityClass() {
        // 抛出异常，防止内部误调用
        throw new AssertionError();
    }
}
```

这样做的缺点是该类不能被子类化，因为所有的构造器都必须显式或隐式地调用超类构造器。

## 避免创建不必要的对象

- 如果对象是不可变的，它始终可以被重用；
- 对于同时提供了静态工厂方法和构造器的不可变类，通常优先使用静态工厂方法，以避免创建不必要的对象。例如，优先使用 `Integer.valueOf(“12”);` 而不是 `new Integer(“12”);` 。构造器的每次调用都会创建一个新的对象，但静态工厂方法不会这样；
- 对于重量级的对象，可以使用对象池；
- 对于廉价的对象，则不需要使用对象池，现代的JVM对小对象的创建和回收动作都是十分廉价的；

## 消除过期的对象引用

造成内存泄漏常见的三种来源：

- 自己管理的内存，一旦元素被释放掉，则该元素中包含的任何对象引用都应该被清空；

  ```java
  public class Stack{
      private Object[] elements;
      private int size = 0;
      private static final int DEFAULT_INITIAL_CAPACITY = 16;
  
      public Stack(){
           elements = new Object[DEFAULT_INITIAL_CAPACITY];
      }
  
      public void push(Object e){
          ensureCapacity();
          elements[size++]=e; // allocate新的堆内存和栈内存
      }
  
      public Object pop(){
          if(size==0) throw new EmptyStackException();
          return element[--size]; // pop出element[size]，该对象不再有效。内存泄漏原因。
      }
  
      private void ensureCapacity(){
          if(elements.length==size)
              elements = Arrays.copyOf(elements, 2*size+1);
      }
  }
  ```

  ```java
  /**
  	在上面的pop方法中，弹出的对象不再有效，但JVM不知道，所以会一直保持该对象，造成内存泄露。
  	解决方法如下：
  */
  	public Object pop(){
          if(size==0) throw new EmptyStackException();
          elements[size] = null; // 等待回收
          return element[--size];
      }
  ```

- 缓存，

- 监听和回调，使用监听和回调要记住取消注册。确保回收的最好的实现是使用弱引用（weak reference），例如，只将他们保存成 `WeakHashMap` 的键。



由于内存泄漏问题通常不会造成明显的失败，因此它可能会在系统中存在很多年。往往只能通过仔细检查代码，或者借助 `Heap剖析工具` 才能发现问题。

## 避免显示调用GC

由于终结方法(`finalizer`)和清除方法(`cleaner`)都不能保证会被及时执行，而且会有非常严重的性能损失，因此应避免调用该方法。

##  优先使用`try-with-resources`

```java
    public static void copy(String src, String dst) {
        try(FileInputStream inputStream = new FileInputStream(src);
            FileOutputStream outputStream = new FileOutputStream(dst)) {
            byte[] buf = new byte[1024];
            int n;
            while ((n = inputStream.read(buf)) > 0) 
                outputStream.write(buf, 0, n);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

使用`try-with-resources`不仅可以使代码更简洁，也更容易进行诊断。

# 所有对象都通用的方法

## 覆盖equals方法遵守通用约定

- 自反性（Reflexive）：对于非`null`的`x`来说，`x.equals(x)`必须返回`true`；
- 对称性（Symmetric）：对于非`null`的`x`和`y`来说，如果`x.equals(y)`为`true`，则`y.equals(x)`也必须为`true`；
- 传递性（Transitive）：对于非`null`的`x`、`y`和`z`来说，如果`x.equals(y)`为`true`，`y.equals(z)`也为`true`，那么`x.equals(z)`也必须为`true`；
- 一致性（Consistent）：对于非`null`的`x`和`y`来说，只要`x`和`y`状态不变，则`x.equals(y)`总是一致地返回`true`或者`false`；
- 对`null`的比较：即`x.equals(null)`永远返回`false`。

## 覆盖equals方法时总要覆盖hashCode

为了保证基于散列的集合使用该类（`HashMap`、`HashSet`、`HashTable`），同时，也是`Object.hashCode`的通用约定，覆盖`equals`方法时，必须覆盖`hashCode`。

- 如果两个对象根据 `equals(object)` 方法比较是相等的，那么这两个对象的 `hashCode`值必须相同；

## 始终要覆盖 toString

Object的toString方法的通用约定是该对象的描述。注意覆盖时，如果有格式，请备注或者严格按照格式返回。

## 谨慎地覆盖clone

对象拷贝的更好的方法是提供一个拷贝构造器或拷贝工厂。

```java
public Type(Type type) {
    // ...
}

public static Type newInstance() {
    // ...
}
```

复制功能最好由构造器或者工厂提供，只有在复制数组时才利用`clone`方法。

## 考虑实现 Comparable 接口

每当要实现一个对排序敏感的类时，都应该实现 `Comparable ` 接口，并且在`compareTo` 方法中比较域值时，要避免使用 `<` 和 `>` ，而应该使用装箱基本类型中的静态方法`compare` 。

# 类和接口



# 泛型



# 枚举和注解



# Function、Lambda 和 Stream

## Lambda 优先于匿名类

Lambda没有名称和文档，因此如果一个计算本身不是自描述的，或者超出了几行，就不要把放入一个Lambda中。对Lambda而言，一行是最理想的，三行是合理的最大极限。

**匿名类和Lambda的比较：**

- Lambda仅限于函数接口；
- 匿名类可以为带有多个抽象方法的接口创建实例；
- Lambda不能获得对自身的引用，它的关键字 this 是指外围实例；

## 方法引用优先于Lambda

方法引用常常会比Lambda表达式更加简洁。只要方法引用更加简洁、清晰，就用方法引用；否则使用Lambda。

```java
Map<String, Integer> map = new HashMap<>();
map.put("apple", 5);
map.merge("apple", 1, Integer::sum);	// 使用方法引用
map.merge("org", 1, (val, incr) -> val + incr);	// 使用Lambda
```

## 坚持使用标准的函数接口

`java.util.Function` 包含了很多的接口，但只要记住其中6个基础接口，就可以推断出其余接口：

- 基础接口作用于对象引用类型；
- `Operator`接口代表其结果与参数类型一致的函数；
- `Predicate`接口代表带有一个参数并返回一个`boolean`的函数；
- `Function`接口代表其参数与返回的类型不一致的函数；
- `Supplier`接口代表没有参数并且返回一个值的函数；
- `Consumer`接口代表的是带有一个函数但不返回任何值的函数；



| 接口                | 函数签名               | 范例 |
| ------------------- | ---------------------- | ---- |
| `UnaryOperator<T>`  | `R apply(T t);`        |      |
| `BinaryOperator<T>` | `T apply(T t1, T t2);` |      |
| `Predicate<T>`      | `boolean test(T t);`   |      |
| `Function<T, R>`    | `R apply(T t);`        |      |
| `Supplier<T>`       | `T get();`             |      |
| `Consumer<T>`       | `void accept(T t);`    |      |

这6中基础接口各自有3中变体，分别作用于基础数据类型int、long和double，它们的命名类如 `IntPredicate`。

`Function` 接口还有9种变体，用于结果类型为基本类型的情况。类如`LongToIntFunction` ，``

还有带两个参数的版本，如`BiFunction<T, U, R>` 、`BiConsumer<T, U>` 、`BiPredicate<T, U>`。

## 谨慎使用 Stream

- 滥用 `Stream` 会使程序的可读性变差；
- `Stream` 支持 `int、long和double`三种基本类型，但不支持`char`，因此应避免利用 `Stream` 处理 char 值；
- 在迭代版的代码块中，可以读取或者修改范围内的任意局部变量；`Stream` 的 Lambda 中只能读取 final 或者有效的final变量，并且不能修改任何局部变量；
- 在迭代版的代码块中，可以从外围方法中 `return、break、continue`外围循环，或抛出异常；`Stream` 的 Lambda 则不能做到；

## 优先选择 Stream 中无副作用的函数

为了获得Stream带来的优势，传入Stream操作的任何函数都应该是纯函数（==函数的输出结果只取决于输入，不依赖于任何可变的状态，也不会更新任何状态==）。

```java
// 不要这样使用 Stream！！！
// 利用一个改变外部状态的 Lambda，完成了在终止操作forEach中的所用工作
Map<String, Long> freq = new HashMap<>();
try(Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word, 1L, Long::sum);
    });
}
```

```java
// 正确的使用 Stream！！！
Map<String, Long> freq = null;
try(Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

> forEach 操作应该只用于报告 Stream 计算的结果，而不是执行计算。

### Collectors

```java
// toMap
toMap(Function<? super T, ? extends K> keyMapper,
      Function<? super T, ? extends U> valueMapper);
toMap(Function<? super T, ? extends K> keyMapper,
      Function<? super T, ? extends U> valueMapper,
      BinaryOperator<U> mergeFunction);
toMap(Function<? super T, ? extends K> keyMapper,
      Function<? super T, ? extends U> valueMapper,
      BinaryOperator<U> mergeFunction,
      Supplier<M> mapSupplier);
// 上面3种的变体，命名为 toConcurrentMap
toConcurrentMap(Function<? super T, ? extends K> keyMapper,
                Function<? super T, ? extends U> valueMapper);
toConcurrentMap(Function<? super T, ? extends K> keyMapper,
                Function<? super T, ? extends U> valueMapper,
                BinaryOperator<U> mergeFunction);
toConcurrentMap(Function<? super T, ? extends K> keyMapper,
                Function<? super T, ? extends U> valueMapper,
                BinaryOperator<U> mergeFunction,
                Supplier<M> mapSupplier);
```



```java
// joining
joining();
joining(CharSequence delimiter);
joining(CharSequence delimiter, CharSequence prefix, CharSequence suffix);
```



```java
// toList
toList();
```



```java
// groupingBy
groupingBy(Function<? super T, ? extends K> classifier);
groupingBy(Function<? super T, ? extends K> classifier,
           Collector<? super T, A, D> downstream);
groupingBy(Function<? super T, ? extends K> classifier,
           Supplier<M> mapFactory,
           Collector<? super T, A, D> downstream);
```

## Stream 要优先用Collection作为返回类型

## 谨慎使用 Stream 并行

在Stream上通过并行获得的性能，最好是通过 `ArrayList`、`HashMap`、`HashSet` 和 `ConcurrentHashMap`实例，数组等。这些数据结构的共性是可以被精确、轻松地分成任意大小的子范围。

应当尽量不要并行 Stream，除非有着足够的自信并在真实环境下进行过性能测试。不恰当的使用，可能会降低程序性能，甚至导致运行失败。




























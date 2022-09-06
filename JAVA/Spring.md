# Spring
## 一、Spring框架概述

1. Spring是轻量级的开源的 JavaEE 框架
2. Spring 有两个核心部分：IOC 和 AOP
   1. IOC：控制反转，把创建对象过程交给Spring进行管理。
   2. AOP：面向切面，不修改源代码就可以功能增强。

3. Spring的特定：
   1. 方便解耦
   2. AOP编程支持
   3. 方便程序测试
   4. 很容易和其他框架整合
   5. 方便进行事务操作
   6. 降低API开发难度

![](https://gitee.com/yanjundong97/picBed/raw/master/images/Spring结构.png)



## 二、IOC容器

### 1、什么是IOC

1. 控制反转（Inversion of Control, IOC），把==对象创建和对象之间的调用过程==，交给Spring进行管理。它可以在对象生成或者初始化时将属性注入，而且这种依赖注入是可以递归的，对象被逐层注入。
2. 使用IOC目的：降低耦合度
3. IOC是一种设计模式，Spring IOC只是其中的一种实现。

### 2、IOC的实现原理

涉及技术：

- XML解析
- 工厂模式
- 反射

![](https://gitee.com/yanjundong97/picBed/raw/master/images/IOC实现.png)

> 使用XML配置文件的方式进一步降低耦合度（相比较`直接创建对象`和`工厂模式创建`）。

### 3、IOC接口

IOC 思想基于IOC容器完成，==IOC 容器底层就是对象工厂==。

Spring 提供 IOC 容器实现两种方式：（两个接口）

- `BeanFactory`：IOC容器的基本实现，是Spring内部的使用接口，不提供开发人员进行使用。
  
  - 特点：加载配置文件时不会创建对象，在获取（使用）对象才去创建对象。
- `ApplicationContext`：`BeanFactory`接口的子接口，提供更多的功能。
  
- 特点：加载配置文件时就会把配置文件中的对象进行创建。
  
- `ApplicationContext` 接口的实现类：

  ![image-20200907164152727](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200907164152727.png)

### 4、IOC操作Bean管理

#### 4.1 Bean管理

`Bean管理` 包括了两个操作：`Spring创建对象` 和 `Spring注入属性` 。

其实现方法可以 `基于XML配置文件方式`，也可以`基于注解方式实现`（见4.6）。

##### 基于XML配置文件实现

1. 创建对象

   ```xml
   <bean id="user" class="com.coder.demo.User" ></bean>
   ```

   > 在Spring配置文件中，使用bean标签，就可以实现bean创建对象。
   >
   > bean标签中常用的属性：
   >
   > 1. id：唯一标识
   > 2. class：类的全路径
   >
   > 创建对象时，默认执行无参构造方法完成对象创建。

2. 注入属性

   DI：依赖注入，就是注入属性

   ```xml
   <!------------使用 setter 方法注入属性------------>
   <bean id="user" class="com.coder.demo.User" >
       <property name="name" value="Jack"></property>
       <property name="phone" value="18135888888"></property>
   </bean>
   
   <!------------使用 有参构造 注入属性------------>
   <bean id="order" class="com.coder.demo.Order">
       <constructor-arg name="ID" value="1001"></constructor-arg>
       <constructor-arg name="money" value="100.9"></constructor-arg>
       <constructor-arg name="address" value="陕西省西安市"></constructor-arg>
   </bean>
   
   <!--------------注入外部bean-------------->·
   <bean name="userDao" class="com.coder.demo.UserDaoImpl"></bean>
   
   <bean name="userService" class="com.coder.demo.UserService">
       <!--通过 ref属性 注入外部bean-->
       <property name="userDao" ref="userDao"></property>
   </bean>
   
   <!--------------集合类型属性注入-------------->
   <bean id="user" class="com.coder.demo.User">
       <property name="name" value="Jack"></property>
       <property name="phone" value="18135908110"></property>
       <!--list类型属性注入-->
       <property name="orders">
           <list>
               <ref bean="order1"></ref>
               <ref bean="order2"></ref>
           </list>
       </property>
       <!--map类型属性注入-->
       <property name="maps">
           <map>
               <entry key="1" value="Java"></entry>
               <entry key="2" value="Python"></entry>
               <entry key="3" value="GoLang"></entry>
           </map>
       </property>
   </bean>
   
   ```
#### 4.2 FactoryBean

`Spring` 有两种类型的bean，一种是普通bean，另一种是工厂bean（`FactoryBean`）。

- 普通bean：在配置文件中定义的bean类型就是返回类型。

  ```java
  <bean id="user" class="com.coder.demo.User"></bean>
  ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
  User bean = context.getBean("user", User.class);
  ```

- 工厂bean：在配置文件中定义的bean类型可以和返回类型不一样。

  ```java
  <bean name="myBean" class="com.coder.demo.factoryBean.MyBean"></bean>
  
  /*
  	创建bean，让这个类作为工厂bean，实现接口FactoryBean
  */
  public class MyBean implements FactoryBean<Order> {
      @Override
      public Order getObject() throws Exception {
          return new Order("1002", 999.99, "陕西省西安市");
      }
  
      @Override
      public Class<?> getObjectType() {
          return Order.class;
      }
  
      @Override
      public boolean isSingleton() {
          return false;
      }
  }
  
  ApplicationContext context = new ClassPathXmlApplicationContext("bean2.xml");
  Order course = context.getBean("myBean", Order.class);
  ```

#### 4.3 bean的作用域

- 在Spring里面，设置创建bean实例是单实例还是多实例。

- 在Spring里，默认情况下，bean是 `单实例对象`。

- 使用配置文件bean标签的属性（scope） 用于设置单实例还是多实例。scope的属性值：

  - `singleton` ，默认值，表示是单例对象；

  - `prototype` ，表示是多实例对象

    ```xml
    <bean id="user" class="com.coder.demo.User" scope="prototype"></bean>
    ```

> `singleton` 和 `prototype` 的区别？
>
> 1. `singleton` 单实例， `prototype` 多实例；
>
> 2. `singleton` ：加载配置文件时就会创建单实例对象；
>
>     `prototype` ：不是在加载配置文件时才创建对象，在调用 `getBean` 方法时创建多实例对象。

#### 4.4 bean生命周期

1. 通过构造器创建 bean 实例（无参数构造）
2. 为 bean 的属性设置值和对其他 bean 的引用（调用 `setter` 方法）
3. 调用 bean 的初始化的方法（需要进行配置初始化的方法） 
4. bean 可以使用了（对象获取到了）
5. 当容器关闭的时候，调用 bean 销毁的方法（需要进行配置销毁的方法）

```java
public class Orders {

    public Orders() {
        System.out.println("第一步---执行无参构造器创建bean实例");
    }

    private String name;

    public void init() {
        System.out.println("第三步---执行初始化方法");
    }

    public void setName(String name) {
        System.out.println("第二步---调用 setter 方法设值属性值");
        this.name = name;
    }

    public void destroy() {
        System.out.println("第五步---执行销毁方法");
    }
}
```



```xml
<bean id="orders" class="com.coder.bean.Orders" init-method="init" 
      destroy-method="destroy">
    <property name="name" value="电脑"></property>
</bean>
```



```java
public class Main1 {

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean3.xml");
        Orders bean = context.getBean("orders", Orders.class);
        System.out.println("第四步---可以使用了：" + bean);

        ((ClassPathXmlApplicationContext)context).close();
    }

}

/**
第一步---执行无参构造器创建bean实例
第二步---调用 setter 方法设值属性值
第三步---执行初始化方法
第四步---可以使用了：com.coder.bean.Orders@dc24521
第五步---执行销毁方法
*/
```

> 加入 bean 的 `后置处理器`后，bean的生命周期有七步：
>
> 1. 通过构造器创建 bean 实例（无参数构造）
> 2. 为 bean 的属性设置值和对其他 bean 的引用（调用 `setter` 方法）
> 3. **把bean实例传递给后置处理器中的 `postProcessBeforeInitialization` 方法**
> 4. 调用 bean 的初始化的方法（需要进行配置初始化的方法） 
> 5. **把bean实例传递给后置处理器中的 `postProcessAfterInitialization` 方法**
> 6. bean 可以使用了（对象获取到了）
> 7. 当容器关闭的时候，调用 bean 销毁的方法（需要进行配置销毁的方法）
>
> ```java
> //创建后置处理器
> public class MyProcess implements BeanPostProcessor {
>     @Override
>     public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
>         System.out.println("初始化前传递给后置处理器");
>         return bean;
>     }
> 
>     @Override
>     public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
>         System.out.println("初始化后传递给后置处理器");
>         return bean;
>     }
> }
> ```
>
> ```xml
> <bean id="orders" class="com.coder.bean.Orders" init-method="init" destroy-method="destroy">
>     
>     
>     <property name="name" value="电脑"></property>
> </bean>
> <!--在配置文件中配置后，该文件中的所有bean实例都会使用该 后置处理器-->
> <bean id="myProcess" class="com.coder.bean.MyProcess"></bean>
> ```
>
> ```java
> /**
> 第一步---执行无参构造器创建bean实例
> 第二步---调用 setter 方法设值属性值
> 初始化前传递给后置处理器
> 第三步---执行初始化方法
> 初始化后传递给后置处理器
> 第四步---可以使用了：com.coder.bean.Orders@7e0b0338
> 第五步---执行销毁方法
> */
> ```
>



#### 4.5 自动装配

`自动装配`：根据指定装配规则（属性名称或者属性类型），Spring自动将匹配的属性值进行注入。

```xml
<!--手动装配-->
<bean id="emp" class="com.coder.autowrie.Emp">
    <property name="dept" ref="dept"></property>
</bean>

<!--通过名称自动装配-->
<bean id="emp1" class="com.coder.autowrie.Emp" autowire="byName"></bean>

<!--通过类型自动装配-->
<bean id="emp2" class="com.coder.autowrie.Emp" autowire="byType"></bean>

<bean id="dept" class="com.coder.autowrie.Dept"></bean>
```

#### 4.6 基于注解方式

使用注解的目的：简化 XML 配置

Spring 针对Bean管理中创建对象提供注解：

- `@Component`
- `@Service`
- `@Controller`
- `@Repository`

> 上面四个注解功能都是一样的，都可以用来创建 Bean 实例。

##### 基于注解的对象创建

1. 引入依赖 `spring-aop-5.2.6.RELEASE.jar`

2. 开启组件扫描

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                              http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
       <!-- 开启组件扫描
    		1. 如果扫描多个包，多个包之间用逗号隔开
   	-->
       <context:component-scan base-package="com.coder.annotation"></context:component-scan>
   </beans>
   ```

3. 创建类，在类上添加注解

   ```java
   /* value属性就相当于 配置文件中的 id
   * 默认情况下，就是类名的首字母变小写后的结果
   *  */
   @Service(value = "userService")
   public class UserService {
   
       public void add() {
           System.out.println("add......");
       }
   
   }
   ```

##### 基于注解的属性注入

- `@Autowired`：根据属性类型进行自动装配

  ```java
  @Service(value = "userService")
  public class UserService {
  
      @Autowired
      private UserDao userDao;
  
      public void insert() {
          System.out.println("service insert...");
          userDao.insert();
      }
  
  }
  ```

- `@Qualifier`：根据属性名称进行注入 ，和上面的 @Autowired 一起使用

  如果 `UserDao` 这个接口有多个实现类，那使用 `@Autowired` 时根据类型注入就不知道使用哪个类型去注入属性，但是可以使用 `@Qualifier` 指定 bean 的名称来注入。

  ```java
  @Service(value = "userService")
  public class UserService {
  
      @Autowired
      //使用 @Qualifier 根据bean的名称来注入属性
      @Qualifier(value = "userDaoDefaultImpl")
      private UserDao userDao;
  
      public void insert() {
          System.out.println("service insert...");
          userDao.insert();
      }
  
  }
  ```

- `@Resource`：可以根据类型或名称注入

  ```java
  @Service(value = "userService")
  public class UserService {
  
      //@Resource //根据类型注入
      @Resource(name = "userDaoDefaultImpl")  //根据属性名称注入
      private UserDao userDao;
  
      public void insert() {
          System.out.println("service insert...");
          userDao.insert();
      }
  
  }
  ```

- `@Value`：注入普通属性类型

##### 完全注解开发

1. 创建配置类，替代xml配置文件

   ```java
   @Configuration
   @ComponentScan(value = "com.coder.annotation")
   public class SpringConfig {
   }
   ```

2. 测试类

   ```java
   public class Main1 {
   
       public static void main(String[] args) {
           
           ApplicationContext context = 
               new AnnotationConfigApplicationContext(SpringConfig.class);
           UserService bean = context.getBean("userService", UserService.class);
           System.out.println(bean);
           bean.insert();
       }
   
   }
   ```
   



## 三、源码解析

### 1、IOC实现的基本原理

```java
/* UserController.java */
public class UserController {

    @AutoWiredd
    private UserService userService;

    public UserService getUserService() {
        return userService;
    }
}

/* UserService.java */
public class UserService {
    
}

/* AutoWiredd.java */
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Target(ElementType.FIELD)
@Documented
public @interface AutoWiredd {
}
```

> 上面的注解不是 `AutoWired` ，而是一个自定义注解。
>
> 下面的代码通过反射实例化对象

```java
public class Test {
    public static void main(String[] args) throws IllegalAccessException, InstantiationException {
        UserController userController = new UserController();
        Class<? extends UserController> clazz = userController.getClass();
        for (Field field : clazz.getDeclaredFields()) {
            AutoWiredd annotation = field.getAnnotation(AutoWiredd.class);
            if (annotation != null) {
                Class<?> type = field.getType();
                Object instance = type.newInstance();
                field.setAccessible(true);
                field.set(userController, instance);
            }
        }

        System.out.println(userController.getUserService());
        //com.coder.ioc.UserService@4617c264

    }
}
```

 ### 2、bean创建过程的代码调式

> 测试代码

```java
public class Main {

    public static void main(String[] args) {
        // 启动时创建对象
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        // 从map 中获取bean
        Order order = context.getBean("order3", Order.class);
        System.out.println(order);
    }
}
```

```java
// 使用这个构造函数初始化 ClassPathXmlApplicationContext
public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
    this(new String[] {configLocation}, true, null);
}
```

> `ClassPathXmlApplicationContext.java` 的构造函数，该类的所有构造函数最终都会调用下面的这个方法。

```java
public ClassPathXmlApplicationContext(
    String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
    throws BeansException {

    super(parent);
    setConfigLocations(configLocations);
    if (refresh) {
        // 所有单实例的 bean 创建完成
        refresh();
    }
}
```

> `AbstractApplicationContext#refresh()` 

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // Prepare this context for refreshing.
        prepareRefresh();

        // Spring解析xml配置文件，将要创建的所有bean配置信息保存起来。可以查看Spring对xml的解析
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // Prepare the bean factory for use in this context.
        prepareBeanFactory(beanFactory);

        try {
            // 设置 BeanFactory 的后置处理器
            postProcessBeanFactory(beanFactory);

            // 调用 BeanFactory 的后置处理器，这些后置处理器是在 Bean 定义中向容器注册的
            invokeBeanFactoryPostProcessors(beanFactory);

            // 注册 Bean 的后置处理器，在 Bean 创建过程中调用
            registerBeanPostProcessors(beanFactory);

            // Initialize message source for this context.
            // 支持国际化功能
            initMessageSource();

            // Initialize event multicaster for this context.
            initApplicationEventMulticaster();

            // 留给子类的方法
            onRefresh();

            // Check for listener beans and register them.
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            // 初始化所有 (non-lazy-init) 单实例bean
            finishBeanFactoryInitialization(beanFactory);

            // 发布容器事件，结束 Refresh 过程
            finishRefresh();
        }

        catch (BeansException ex) {
            // 防止 Bean 资源占用，销毁之前创建的 单实例bean
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
        }
    }
}
```

![image-20201105104524120](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201105104524120.png)

#### 2.1 BeanDefinition的载入

> `AbstractApplicationContext#obtainFreshBeanFactory` 

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    refreshBeanFactory();
    return getBeanFactory();
}
```



> `AbstractRefreshableApplicationContext#refreshBeanFactory` ，先构建了一个 IOC容器（`DefaultListableBeanFactory`）供 `ApplicationContext` 使用。同时启动 `loadBeanDefinitions` 来加载  `BeanDefinition`信息

```java
protected final void refreshBeanFactory() throws BeansException {
    // 如果已经创建了 BeanFactory，则销毁并关闭该 BeanFactory
    if (hasBeanFactory()) {
        destroyBeans();
        closeBeanFactory();
    }
    try {
        // 这里的 createBeanFactory 会 new 一个 DefaultListableBeanFactory 对象
        DefaultListableBeanFactory beanFactory = createBeanFactory();
        beanFactory.setSerializationId(getId());
        customizeBeanFactory(beanFactory);
        // 通过一个 XmlBeanDefinitionReader 来载入 BeanDefinition 信息
        loadBeanDefinitions(beanFactory);
        synchronized (this.beanFactoryMonitor) {
            this.beanFactory = beanFactory;
        }
    }
    catch (IOException ex) {
        throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
    }
}


protected DefaultListableBeanFactory createBeanFactory() {
    return new DefaultListableBeanFactory(getInternalParentBeanFactory());
}
```

> `AbstractXmlApplicationContext#loadBeanDefinitions(DefaultListableBeanFactory)` 通过 beanFactory 实例化一个 `XmlBeanDefinitionReader`，通过它 来载入 BeanDefinition 信息。
>
> `AbstractXmlApplicationContext#loadBeanDefinitions(XmlBeanDefinitionReader)`  ，首先得到`BeanDefinition`信息的 `Resource` 定位，接着通过`XmlBeanDefinitionReader` 来读取，具体的载入过程是委托给 `AbstractBeanDefinitionReader` 来完成。

```java
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
    // Create a new XmlBeanDefinitionReader for the given BeanFactory.
    XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

    // Configure the bean definition reader with this context's
    // resource loading environment.
    beanDefinitionReader.setEnvironment(this.getEnvironment());
    beanDefinitionReader.setResourceLoader(this);
    beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

    // Allow a subclass to provide custom initialization of the reader,
    // then proceed with actually loading the bean definitions.
    initBeanDefinitionReader(beanDefinitionReader);
    loadBeanDefinitions(beanDefinitionReader);
}

protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
    // 以 Resource 的方式来获得配置文件的资源位置
    Resource[] configResources = getConfigResources();
    if (configResources != null) {
        reader.loadBeanDefinitions(configResources);
    }
    // 以 String 的方式来获得配置文件的位置
    String[] configLocations = getConfigLocations();
    if (configLocations != null) {
        // 使用 XmlBeanDefinitionReader 来加载 beanDefinitions
        reader.loadBeanDefinitions(configLocations);
    }
}
```

> `AbstractBeanDefinitionReader#loadBeanDefinitions ` 

```java
public int loadBeanDefinitions(String location, @Nullable Set<Resource> actualResources) throws BeanDefinitionStoreException {
    // 这里取得 ResourceLoader，使用的是 ClassPathXmlApplicationContext
    ResourceLoader resourceLoader = getResourceLoader();
    if (resourceLoader == null) {
        throw new BeanDefinitionStoreException(
            "Cannot load bean definitions from location [" + location + "]: no ResourceLoader available");
    }

    // 这里对 resource 的路径模式进行解析，比如我们设置好的各种ant格式的路径，得到需要的 Resource 集合 
    if (resourceLoader instanceof ResourcePatternResolver) {
        // Resource pattern matching available.
        try {
            // 获取 Resource 的定位信息，详见 2.1.1
            Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
            // 解析xml文件来加载 beanDefinition，详见 2.1.2
            int count = loadBeanDefinitions(resources);
            if (actualResources != null) {
                Collections.addAll(actualResources, resources);
            }
            return count;
        }
        catch (IOException ex) {
            throw new BeanDefinitionStoreException(
                "Could not resolve bean definition resource pattern [" + location + "]", ex);
        }
    }
    else {
        // Can only load single resources by absolute URL.
        Resource resource = resourceLoader.getResource(location);
        int count = loadBeanDefinitions(resource);
        if (actualResources != null) {
            actualResources.add(resource);
        }
        return count;
    }
}
```



##### 2.1.1 Resource定位过程

> `DefaultResourceLoader#getResource` 取得 Resource 的具体过程，先判断是否以 `/` 开头，再判断处理以`classpath:`  开头的 Resource，再处理 URL 标识的 Resource定位，如果都不是 就把任务交 给 `getResourceByPath` 方法，默认实现是得到一个 `ClassPathContextResource` 。

```java
@Override
public Resource getResource(String location) {
    Assert.notNull(location, "Location must not be null");

    for (ProtocolResolver protocolResolver : getProtocolResolvers()) {
        Resource resource = protocolResolver.resolve(location, this);
        if (resource != null) {
            return resource;
        }
    }

    if (location.startsWith("/")) {
        return getResourceByPath(location);
    }
    else if (location.startsWith(CLASSPATH_URL_PREFIX)) {
        return new ClassPathResource(location.substring(CLASSPATH_URL_PREFIX.length()), getClassLoader());
    }
    else {
        try {
            // Try to parse the location as a URL...
            URL url = new URL(location);
            return (ResourceUtils.isFileURL(url) ? new FileUrlResource(url) : new UrlResource(url));
        }
        catch (MalformedURLException ex) {
            // No URL -> resolve as resource path.
            return getResourceByPath(location);
        }
    }
}


/**
	 * Return a Resource handle for the resource at the given path.
	 * <p>The default implementation supports class path locations. This should
	 * be appropriate for standalone implementations but can be overridden,
	 * e.g. for implementations targeted at a Servlet container.
	 */
protected Resource getResourceByPath(String path) {
    return new ClassPathContextResource(path, getClassLoader());
}
```

##### 2.1.2 BeanDefinition的载入

由于这里使用的是 XML 方式的定义，所有需要使用 `XmlBeanDefinitionReader` 。如果使用了其他的 `BeanDefinition`方式，就需要使用其他种类的 `BeanDefinitionReader` 来完成数据的载入工作。

> `AbstractBeanDefinitionReader#loadBeanDefinitions` 

```java
@Override
public int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException {
    Assert.notNull(resources, "Resource array must not be null");
    int count = 0;
    for (Resource resource : resources) {
        // 这个方法在 AbstractBeanDefinitionReader 中并没有实现，具体的实现在 XmlBeanDefinitionReader 中
        count += loadBeanDefinitions(resource);
    }
    return count;
}
```

> `XmlBeanDefinitionReader` 

```java
@Override
public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
    return loadBeanDefinitions(new EncodedResource(resource));
}


public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
    Assert.notNull(encodedResource, "EncodedResource must not be null");

    Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();

    if (!currentResources.add(encodedResource)) {
        throw new BeanDefinitionStoreException(
            "Detected cyclic loading of " + encodedResource + " - check your import definitions!");
    }

    // 这里得到 xml 文件，并得到IO的InputSource准备进行读取
    try (InputStream inputStream = encodedResource.getResource().getInputStream()) {
        InputSource inputSource = new InputSource(inputStream);
        if (encodedResource.getEncoding() != null) {
            inputSource.setEncoding(encodedResource.getEncoding());
        }
        return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
    }
    catch (IOException ex) {
        throw new BeanDefinitionStoreException(
            "IOException parsing XML document from " + encodedResource.getResource(), ex);
    }
    finally {
        currentResources.remove(encodedResource);
        if (currentResources.isEmpty()) {
            this.resourcesCurrentlyBeingLoaded.remove();
        }
    }
}


/**
 	真正从指定的xml文件中加载 bean definitions
*/
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
    throws BeanDefinitionStoreException {

    try {
        // 取得xml文件的 Document 对象
        Document doc = doLoadDocument(inputSource, resource);
        // 对 BeanDefinition 解析的详细过程
        int count = registerBeanDefinitions(doc, resource);
        return count;
    }
    // 异常处理
    ...
}

/**
	
*/
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
    // 这里得到BeanDefinitionDocumentReader来对xml的BeanDefinition进行解析，获取的是 DefaultBeanDefinitionDocumentReader
    BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
    int countBefore = getRegistry().getBeanDefinitionCount();
    // 具体的解析过程
    documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
    return getRegistry().getBeanDefinitionCount() - countBefore;
}
```



> `DefaultBeanDefinitionDocumentReader#processBeanDefinition` 。首先完成对 `BeanDefinition` 的处理，处理的结果由 `BeanDefinitionHolder` 对象来持有，这个对象除了持有 `BeanDefinition` 对象外，还持有其他与 `BeanDefinition` 的使用相关的信息。

```java
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
    BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
    if (bdHolder != null) {
        bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
        try {
            // Register the final decorated instance.
            BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
        }
        catch (BeanDefinitionStoreException ex) {
            getReaderContext().error("Failed to register bean definition with name '" +
                                     bdHolder.getBeanName() + "'", ele, ex);
        }
        // Send registration event.
        getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
    }
}
```

![image-20201112164327810](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201112164327810.png)

####  2.2 BeanDefinition在IOC容器中的注册

此时，IOC容器已经完成了管理Bean对象的数据准备工作，但是重要的依赖注入还没有完成，现在，在IOC容器中的`BeanDefinition` 中存在的还只是一些静态的配置信息。这时候的容器还没有完全起作用，要完全发挥容器的作用，还需要完成数据向容器的注册。

> `BeanDefinitionReaderUtils#registerBeanDefinition` 把定义的 `BeanDefinition` 载入到 IOC容器中

```java
public static void registerBeanDefinition(
    BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
    throws BeanDefinitionStoreException {

    // Register bean definition under primary name.
    String beanName = definitionHolder.getBeanName();
    registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

    // Register aliases for bean name, if any.
    String[] aliases = definitionHolder.getAliases();
    if (aliases != null) {
        for (String alias : aliases) {
            registry.registerAlias(beanName, alias);
        }
    }
}

```

> `DefaultListableBeanFactory#registerBeanDefinition` 。在 `DefaultListableBeanFactory` 中，是通过一个 HashMap 来持有载入的 `BeanDefinition` 。
>
> 在向HashMap 中存放 `BeanDefinition` 时，如果遇到同名的，需要依据 `allowBeanDefinitionOverriding` 的配置来处理。

```java
/** Map of bean definition objects, keyed by bean name. */
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);

/** List of bean definition names, in registration order. */
private volatile List<String> beanDefinitionNames = new ArrayList<>(256);

public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
    throws BeanDefinitionStoreException {

    Assert.hasText(beanName, "Bean name must not be empty");
    Assert.notNull(beanDefinition, "BeanDefinition must not be null");

    if (beanDefinition instanceof AbstractBeanDefinition) {
        try {
            ((AbstractBeanDefinition) beanDefinition).validate();
        }
        catch (BeanDefinitionValidationException ex) {
            throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
                                                   "Validation of bean definition failed", ex);
        }
    }

    // 这里检查是否有同名的 BeanDefinition 已经在IOC容器中注册，如果是，又不允许重复，就会抛出异常
    BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
    if (existingDefinition != null) {
        if (!isAllowBeanDefinitionOverriding()) {
            throw new BeanDefinitionOverrideException(beanName, beanDefinition, existingDefinition);
        }
        this.beanDefinitionMap.put(beanName, beanDefinition);
    }
    else {
        // 正常注册 BeanDefinition 的过程
        if (hasBeanCreationStarted()) {
            // Cannot modify startup-time collection elements anymore (for stable iteration)
            synchronized (this.beanDefinitionMap) {
                this.beanDefinitionMap.put(beanName, beanDefinition);
                List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
                updatedDefinitions.addAll(this.beanDefinitionNames);
                updatedDefinitions.add(beanName);
                this.beanDefinitionNames = updatedDefinitions;
                removeManualSingletonName(beanName);
            }
        }
        else {
            // Still in startup registration phase
            this.beanDefinitionMap.put(beanName, beanDefinition);
            this.beanDefinitionNames.add(beanName);
            removeManualSingletonName(beanName);
        }
        this.frozenBeanDefinitionNames = null;
    }

    if (existingDefinition != null || containsSingleton(beanName)) {
        resetBeanDefinition(beanName);
    }
    else if (isConfigurationFrozen()) {
        clearByTypeCache();
    }
}
```

完成了 `BeanDefinition` 的注册，就完成了 IOC 容器的初始化过程，此时在使用的 IOC 容器 `DefaultListableBeanFactory` 中已经建立了整个 Bean的配置信息，而且这些 `BeanDefinition`  已经在 `beanDefinitionMap` 里可以被检索和使用。容器的作用就是对这些信息进行处理和维护。这些信息是容器建立依赖反转的基础。

### 3、IOC容器的依赖注入

> `AbstractApplicationContext#finishBeanFactoryInitialization` ，初始化所有单实例bean

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // Initialize conversion service for this context.
    if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
        beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
        beanFactory.setConversionService(
            beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
    }

    // Register a default embedded value resolver if no bean post-processor
    // (such as a PropertyPlaceholderConfigurer bean) registered any before:
    // at this point, primarily for resolution in annotation attribute values.
    if (!beanFactory.hasEmbeddedValueResolver()) {
        beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
    }

    // Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
    String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
    for (String weaverAwareName : weaverAwareNames) {
        getBean(weaverAwareName);
    }

    // Stop using the temporary ClassLoader for type matching.
    beanFactory.setTempClassLoader(null);

    // Allow for caching all bean definition metadata, not expecting further changes.
    beanFactory.freezeConfiguration();

    // Instantiate all remaining (non-lazy-init) singletons.
    // 真正初始化所有单实例
    beanFactory.preInstantiateSingletons();
}
```



> `DefaultListableBeanFactory#preInstantiateSingletons`

```java
public void preInstantiateSingletons() throws BeansException {
    
    // 拿到所有要创建的 bean 的id
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

    // Trigger initialization of all non-lazy singleton beans...
    // 按顺序创建bean
    for (String beanName : beanNames) {
        // 根据 bean 的id 拿到bean的定义信息【从xml文件中解析获得】，是否单实例、是否工厂bean、是否懒加载等
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        // 判断该bean 【不是抽象】 且 【是单例】 且 【不是懒加载】
        if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
            if (isFactoryBean(beanName)) {
               // 不是工厂bean
                ...
            }
            else {
                getBean(beanName);
            }
        }
    }

    // Trigger post-initialization callback for all applicable beans...
    for (String beanName : beanNames) {
        Object singletonInstance = getSingleton(beanName);
        if (singletonInstance instanceof SmartInitializingSingleton) {
            final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
            if (System.getSecurityManager() != null) {
                AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
                    smartSingleton.afterSingletonsInstantiated();
                    return null;
                }, getAccessControlContext());
            }
            else {
                smartSingleton.afterSingletonsInstantiated();
            }
        }
    }
}
```

![image-20201105111131115](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201105111131115.png)

> `AbstractBeanFactory#getBean` 是一个重载方法，但该方法最终都会调用 `AbstractBeanFactory#doGetBean` 方法

```java
@Override
public Object getBean(String name) throws BeansException {
    return doGetBean(name, null, null, false);
}

/**
	 * Return an instance, which may be shared or independent, of the specified bean.
	 * @param name the name of the bean to retrieve
	 * @param requiredType the required type of the bean to retrieve
	 * @param args arguments to use when creating a bean instance using explicit arguments
	 * (only applied when creating a new instance as opposed to retrieving an existing one)
	 * @param typeCheckOnly whether the instance is obtained for a type check,
	 * not for actual use
	 * @return an instance of the bean
	 * @throws BeansException if the bean could not be created
	 */
@SuppressWarnings("unchecked")
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
                          @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

    final String beanName = transformedBeanName(name);
    Object bean;

    // Eagerly check singleton cache for manually registered singletons.
    // 先从已经注册的所有单实例bean中看有没有
    Object sharedInstance = getSingleton(beanName);
    if (sharedInstance != null && args == null) {
        bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
    }

    else {
        // Fail if we're already creating this bean instance:
        // We're assumably within a circular reference.
        if (isPrototypeCurrentlyInCreation(beanName)) {
            throw new BeanCurrentlyInCreationException(beanName);
        }

        // 这里对 IOC 容器中的 BeanDefinition 是否存在进行检查，检查是否能在当前的 BeanFactory中得到需要的 bean，如果在当前的 BeanFactory 中取不到，就到双亲 BeanFactory 中去取；如果双亲工厂还找不到，就顺着双亲BeanFactory 链一直向上查找
        // Check if bean definition exists in this factory.
        BeanFactory parentBeanFactory = getParentBeanFactory();
        if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
            ...
        }

        // 把这个bean标记为已创建
        if (!typeCheckOnly) {
            markBeanAsCreated(beanName);
        }

        try {
            final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
            checkMergedBeanDefinition(mbd, beanName, args);

            // Guarantee initialization of beans that the current bean depends on.
            // 拿到创建该bean之前需要提前创建好的bean，如果有 就循环创建
            // 还需要检查是否存在循环依赖，如果有就抛出异常
            String[] dependsOn = mbd.getDependsOn();
            if (dependsOn != null) {
                for (String dep : dependsOn) {
						if (isDependent(beanName, dep)) {
							throw new BeanCreationException(...);
						}
						registerDependentBean(dep, beanName);
						try {
							getBean(dep);
						}
						catch (NoSuchBeanDefinitionException ex) {
							throw new BeanCreationException(...);
						}
					}
            }

            // 这里通过调用 createBean 来创建单实例 bean，这里有一个回调函数，会在函数中调用 ObjectFactory 的 createBean
            // Create bean instance.
            if (mbd.isSingleton()) {
                sharedInstance = getSingleton(beanName, () -> {
                    try {
                        return createBean(beanName, mbd, args);
                    }
                    catch (BeansException ex) {
                       
                    }
                });
                bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
            }
            
            ...
        }
        catch (BeansException ex) {
            cleanupAfterBeanCreationFailure(beanName);
            throw ex;
        }
    }

    // Check if required type matches the type of the actual bean instance.
    // 这里对创建的bean进行类型检查，如果没有问题，就返回这个 bean，这个bean是已经包含了依赖关系的 bean
    if (requiredType != null && !requiredType.isInstance(bean)) {
        ...
    }
    return (T) bean;
}
```



> `DefaultSingletonBeanRegistry#getSingleton`

```java
/** Cache of singleton objects: bean name to bean instance. */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

/**
	 * Return the (raw) singleton object registered under the given name,
	 * creating and registering a new one if none registered yet.
	 * @param beanName the name of the bean
	 * @param singletonFactory the ObjectFactory to lazily create the singleton
	 * with, if necessary
	 * @return the registered singleton object
	 */
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(beanName, "Bean name must not be null");
    synchronized (this.singletonObjects) {
        // 先从一个地方【singletonObjects】获取这个bean
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null) {
            beforeSingletonCreation(beanName);
            boolean newSingleton = false;
            boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
            if (recordSuppressedExceptions) {
                this.suppressedExceptions = new LinkedHashSet<>();
            }
            try {
                // 创建bean
                singletonObject = singletonFactory.getObject();
                newSingleton = true;
            }
            catch (IllegalStateException ex) {
                
            }
            catch (BeanCreationException ex) {
                
            }
            finally {
                afterSingletonCreation(beanName);
            }
            if (newSingleton) {
                // 添加创建的 bean
                addSingleton(beanName, singletonObject);
            }
        }
        return singletonObject;
    }
}

protected void addSingleton(String beanName, Object singletonObject) {
    synchronized (this.singletonObjects) {
        // 创建好的对象最终会放到 map 中
        this.singletonObjects.put(beanName, singletonObject);
        this.singletonFactories.remove(beanName);
        this.earlySingletonObjects.remove(beanName);
        this.registeredSingletons.add(beanName);
    }
}
```

![image-20201105113824732](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201105113824732.png)

### 4、BeanFactory和ApplicationContext

![image-20201105115808860](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201105115808860.png)

![image-20201105174657995](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201105174657995.png)

> [上面图片的地址](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201105174657995.png)

上面的接口系统是以 `BeanFactory` 和 `ApplicationContext` 为核心的，首先 `BeanFactory` 是IOC容器中最基本的接口，定义了简单IOC容器的基本功能，而在`ApplicationContext` 的设计中，一方面，它继承了 `BeanFactory` 接口体系中的`ListableBeanFactory` 和 `HierarchicalBeanFactory` ，具备了IOC容器的基本功能；另一方面，通过集成 `MessageSource` 、`ResourceLoader`和`ApplicationEventPublisher` 等接口，`ApplicationContext` 有了比 `BeanFactory`更高级的IOC容器特性。

### 5、BeanFactory 和 FactoryBean

用户在使用容器时，可以使用转义符 `&` 来得到 `FactoryBean` 本身，用来区分通过容器来获取 `FactoryBean` 产生的对象和获取 `FactoryBean` 本身。【`FactoryBean` 是一个 `Bean`，`BeanFactory` 是IOC容器】。举例来说，如果 `myJndiObject` 是一个 `FactoryBean`，那么使用 `&myJndiObject` 得到的是 `FactoryBean` ，而不是 `myJndiObject` 这个 `FactoryBean` 产生出来的对象。

![](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201112201729086.png)

在 `Spring`中，所有的 bean 都是由 `BeanFactory ` 来进行管理的，但对于 `FactoryBean` 而言，这个 `Bean` 不是一个简单的 `Bean`，而是一个能够产生或者修饰对象生成的工厂 `Bean`，它的实现与设计模式中的工厂模式和修饰器模式类似。

## 四、AOP

### 1、什么是AOP

- 面向切面（方面）编程，利用 AOP 可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。
- 通俗描述：在不修改源代码的情况下，在主干功能中添加新的功能。
- 例如，在代码中引入 `Spring Security` 包，不需要修改之间的代码，就可以实现权限管理。

### 2、AOP体系结构

![image-20201113094925984](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201113094925984.png)

最高层是语言和开发环境，`基础（base）` 可以视为待增强对象或者说目标对象；`切面（Aspect）` 通常包含对于基础的增强应用；`配置（configuration）` 可以看成是一种编织，通过在AOP体系中提供这个配置环境，可以把基础和切面结合起来，从而完成切面对目标对象的编织实现。

第二个层次是为语言和开发环境提供支持的，在这个层次中可以看到AOP框架的高层实现，主要包括配置和编织实现两部分内容。

最底层是编织的具体实现模块，图中的各种技术都可以作为编织逻辑的具体实现方法。`Spring AOP`  使用的是Java本身的语言特性，如`Java Proxy代理类`、`拦截器等`技术，来完成AOP编织的实现。

### 3、实现原理

 AOP 底层使用动态代理，有两种情况动态代理：

- 有接口情况，使用 JDK 动态代理，创建接口实现类代理对象，增强类的方法。

  ![image-20200919211308635](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200919211308635.png)

- 没有接口情况，使用 `CGLIB` 动态代理，创建子类的代理对象，增强类的方法。

  ![image-20200919211333942](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200919211333942.png)

### 3、AOP（JDK动态代理）

使用 JDK 动态代理，使用 Proxy 类里面的方法创建代理对象

![image-20200919211653161](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200919211653161.png)

![image-20200919211715552](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200919211715552.png)

该方法的3个参数：

- `loader`：类加载器；
- `interfaces`：增强方法所在的类，这个类实现的接口，支持多个接口；
- `InvocationHandler`：实现这个接口 `InvocationHandler`，创建代理对象，写增强的方法；

```java
/** UserDao.java */
public interface UserDao {
    public int add(int a, int b);
}

/** UserDaoImpl.java */
public class UserDaoImpl implements UserDao {
    @Override
    public int add(int a, int b) {
        System.out.println("执行...");
        return a + b;
    }
}

/** JDKProxy.java */
public class JDKProxy {

    public static void main(String[] args) {
        UserDaoImpl userDao = new UserDaoImpl();
        Class<?>[] intfs = userDao.getClass().getInterfaces();

        UserDao dao =
                (UserDao) Proxy.newProxyInstance(JDKProxy.class.getClassLoader(),
                intfs, new InvocationHandler() {
            //增强的逻辑
            @Override
            public Object invoke(Object proxy, Method method, Object[] args)
                    throws Throwable {
                // 方法之前
                System.out.println(method.getName() + "方法开始执行");

                // 被增强的方法执行
                Object invoke = method.invoke(userDao, args);
                // 方法之后
                System.out.println(method.getName() + "方法结束执行");
                return invoke;
            }
        });

        int add = dao.add(1, 2);
        System.out.println("结果为" + add);
    }

}

/** 输出结果 */
/**
add方法开始执行
执行...
add方法结束执行
结果为3
*/
```

### 4、AOP 术语

1. `连接点`：类里面哪些方法可以被增强，这些方法称为连接点。
2. `切入点（PointCut）`：实际中被真正增强的方法，称为切入点。
3. `通知（增强）（Advice）`
   1. 实际增强的逻辑部分称为通知；
   2. 通知的类型有：==前置通知、后置通知、环绕通知、异常通知、最终通知==

4. `切面`：是一个动作，是把通知应用到切入点的过程。

### 5、AOP操作

1. Spring框架一般都是基于 AspectJ 实现 AOP 操作；

   AspectJ 不是Spring的组成部分，独立 AOP 框架，但一般把 AOP 和 Spring 框架一起使用，进行AOP操作。

2. 基于 AspectJ 实现 AOP 操作，包括`基于XML配置文件实现`、`基于注解方法实现`。

#### 实际操作

1. 在项目中引入 AOP 相关依赖

   ![image-20200923212637461](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200923212637461.png)

2. 切入点表达式
   - 作用：知道对那个类里面的哪个方法进行增强
   - 语法结构：execution(\[权限修饰符]\[返回类型]\[类全路径]\[方法名称]([参数列表])）
   - 举例：`execution(* com.coder.dao.UserDao.add(..))` 对`com.coder.dao.UserDao`类里的add方法进行增强

#### AspectJ注解

1. 创建 `被增强类`

   ```java
   @Component
   public class User {
   
       public void add(){
           System.out.println("add......");
       }
   
   }
   ```

2. 创建 `增强类`，在该类中，在通知方法上添加通知类型注解，使用切入点表达式配置

   ```java
   @Component
   @Aspect
   public class UserProxy {
   
       /** 前置通知 */
       @Before(value = "execution(* com.coder.aspectJ.annotation.User.add())")
       public void doBefore() {
           System.out.println("before.....");
       }
   
       /** 最终通知 */
       @After(value = "execution(* com.coder.aspectJ.annotation.User.add())")
       public void doAfter() {
           System.out.println("after.....");
       }
   
       /** 后置通知 */
       @AfterReturning(value = "execution(* com.coder.aspectJ.annotation.User.add())")
       public void doAfterReturning() {
           System.out.println("after returning.....");
       }
   
       /** 异常通知 */
       @AfterThrowing(value = "execution(* com.coder.aspectJ.annotation.User.add())")
       public void afterThrowing() {
           System.out.println("AfterThrowing.....");
       }
   
       /** 环绕通知 */
       @Around(value = "execution(* com.coder.aspectJ.annotation.User.add())")
       public void doAround(ProceedingJoinPoint joinPoint) throws Throwable {
           System.out.println("around before.....");
   
           /*被增强的方法执行*/
           joinPoint.proceed();
   
           System.out.println("around after.....");
       }
   
   }
   ```

3. 在Spring配置文件中开启组件扫描和生成代理对象

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:aop="http://www.springframework.org/schema/aop"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                              http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                              http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
   
       <!-- 开启组件扫描 -->
       <context:component-scan base-package="com.coder.aspectJ"></context:component-scan>
   
       <!--开启 Aspect 生成代理对象-->
       <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
   </beans>
   ```

> `AfterReturning`：在方法执行后执行，如果有异常就不执行。
>
> `After`：无论有没有异常都会执行。
>
> `AfterThrowing`：在发生异常时才会执行。

##### 相同切入点的抽取

```java
@Component
@Aspect
public class UserProxy {

    /**相同切入点抽取*/
    @Pointcut(value = "execution(* com.coder.aspectJ.annotation.User.add())")
    public void pointDemo() {}

    /** 前置通知 */
    @Before(value = "pointDemo()")
    public void doBefore() {
        System.out.println("before.....");
    }
}
```

##### 多个增强类

有多个增强类对同一个方法进行增强，可以通过`@Order(数字类型值)` 设置增强类优先级，数字越小优先级越高。

#### AspectJ配置文件



## 五、JdbcTemplate

### 1、JdbcTemplate操作数据库

1. 引入相关jar包

![image-20201003213747760](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201003213747760.png)

2. 在 spring 配置文件中配置数据库连接池

   ```xml
   <!--数据库连接池-->
   <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
       <property name="url" value="jdbc:mysql://192.168.1.149:3306/learn_spring" />
       <property name="username" value="root" />
       <property name="password" value="xidian" />
       <property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
   </bean>
   ```

   

3. 配置 `JdbcTemplate` 对象，注入 `DataSource`。

   ```xml
   <!-- JdbcTemplate 对象-->
   <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
       <property name="dataSource" ref="dataSource" />
   </bean>
   ```

   

4. 创建service类，创建dao类，在dao中注入  `JdbcTemplate`  对象。

   ```java
   public interface BookDao {
   
       public Integer addBook(Book book);
   
   }
   
   @Repository
   public class BookDaoImpl implements BookDao {
       @Autowired
       private JdbcTemplate jdbcTemplate;
   
       @Override
       public Integer addBook(Book book) {
           String sql = "insert into book(id, name, price) values (? ,?, ?)";
           int update = jdbcTemplate.update(sql, book.getId(), book.getName(), book.getPrice());
           return update;
       }
   }
   
   @Service
   public class BookService {
   
       @Autowired
       private BookDao bookDao;
   
       public void addBook(Book book) {
           Integer insert = bookDao.addBook(book);
           System.out.println(insert);
       }
   }
   ```

5. 测试

   ```java
   public class Main4 {
   
       public static void main(String[] args) {
           ApplicationContext context = 
                   new ClassPathXmlApplicationContext("bean7.xml");
           BookService bean = context.getBean(BookService.class);
           Book book = new Book(1000, "Java核心技术", 100);
           bean.addBook(book);
       }
   
   }
   ```

## 六、事务 

### 1、事务

ACID：

- 原子性
- 一致性
- 隔离性
- 永久性

### 2、事务管理

在Spring中进行事务管理操作有两种方式：编程式事务管理、声明式事务管理。

在Spring中进行 `声明式事务管理（包括注解方式、xml配置文件方式）`，底层使用 AOP 原理。

Spring事务管理API，提供了一个接口，代表事务管理器，这个接口针对不同的框架提供了不同的实现类，

![image-20201004222643306](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201004222643306.png)

#### 2.1 基于注解实现事务管理

1. 配置事务

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!--数据库连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="url" value="jdbc:mysql://192.168.1.149:3306/learn_spring" />
        <property name="username" value="root" />
        <property name="password" value="xidian" />
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
    </bean>

    <!-- JdbcTemplate 对象-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource" />
    </bean>

    <!-- 开启组件扫描 -->
    <context:component-scan base-package="com.coder"></context:component-scan>

    <!--创建事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--注入数据源-->
        <property name="dataSource" ref="dataSource" />
    </bean>

    <!--开启事务注解-->
    <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>

</beans>
```

2. 在service类上面（或者service类里面方法上面）添加事务注解`@Transactional`

#### 2.2 事务操作（参数配置）

![image-20201004224206233](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201004224206233.png)

1. `propagation` ：事务的传播行为

   多事务方法直接进行调用，这个过程中事务是如何进行管理的

   ![image-20201004223919417](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201004223919417.png)

   ![image-20201004224005016](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201004224005016.png)

2. `isolation`：事务隔离级别

    事务有特征成为隔离性，多事务操作之间不会产生影响。不考虑隔离性会产生很多问题。

   3个读问题：脏读、不可重复读、虚（幻）读。

   > 脏读：一个未提交事务读取到另一个未提交事务的数据。
   >
   > 不可重复读：
   >
   > 幻读：一个未提交事务读取到另一个提交事务添加数据
   >
   > 通过设置事务隔离级别，可以解决读问题：
   >
   > ![image-20201004230841028](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201004230841028.png)
   
3. `timeout`：超时时间。事务需要在一定的时间内进行提交，否则就要进行回滚。（默认为-1）
4. `readOnly`：是否只读
5. `rollbackFor`：回滚。设置出现哪些异常进行事务回滚。
6. `noRollbackFor`：不回滚。设置出现哪些异常不进行事务回滚。

#### 2.3 完全注解方式

```java
@Configuration
@EnableTransactionManagement
@ComponentScan(value = "com.coder.tx")
public class TxConfig {

    @Bean
    public DruidDataSource getDataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUrl("jdbc:mysql://192.168.1.149:3306/learn_spring");
        dataSource.setUsername("root");
        dataSource.setPassword("xidian");
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        return dataSource;
    }

    @Bean
    public JdbcTemplate getJdbcTemplate(DataSource dataSource) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        jdbcTemplate.setDataSource(dataSource);
        return jdbcTemplate;
    }

    @Bean
    public DataSourceTransactionManager getDataSourceTxManager(DataSource dataSource) {
        DataSourceTransactionManager dataSourceTxManager = new DataSourceTransactionManager();
        dataSourceTransactionManager.setDataSource(dataSource);
        return dataSourceTransactionManager;
    }

}
```



```java
/*测试*/
public class Main5 {
    public static void main(String[] args) {
        ApplicationContext context =
                new AnnotationConfigApplicationContext(TxConfig.class);
        AccountService bean = context.getBean(AccountService.class);
        bean.editMoney();
    }
}
```

# SpringMVC

## 一、概述

1. Spring为展示层提供的基础 MVC 设计理念的优秀Web框架，是目前最主流的MVC框架之一；
2. Spring MVC 通过一套MVC注解，让POJO成为处理请求的控制器，而无须实现任何接口。
3. 支持REST风格的URL请求
4. 采用了松散耦合可插拔组件结构，比其他MVC框架更具扩展性和灵活性

![image-20201027102453740](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201027102453740.png)

### 1、HelloWorld

```xml
<!-- web.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!-- 配置DispatherServlet的初始化参数：设置文件的路径和文件名称 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/applicationContext.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

```xml
<!-- dispatcher-servlet.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 设置扫描组件的包 -->
    <context:component-scan base-package="com.coder" />

    <!--指定视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- 视图的路径 -->
        <property name="prefix" value="/WEB-INF/pages/"/>
        <!-- 视图名称后缀  -->
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

```java
/* Hello.java */
@Controller
@RequestMapping("/hi")
public class HelloController {

    @RequestMapping("/hello")
    public String hello() {
        return "hello";
    }

}
```

```jsp
<%-- hello.jsp --%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h1>Hello...</h1>
</body>
</html>
```

### 2、处理流程

![image-20201027165110314](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201027165110314.png)

1)  客户端请求提交到**DispatcherServlet**

2)  由`DispatcherServlet` 控制器查询一个或多个**HandlerMapping**，找到处理请求的**Controller**

3)  DispatcherServlet将请求提交到Controller（也称为Handler）

4)  Controller调用业务逻辑处理后，返回**ModelAndView**

5)  DispatcherServlet查询一个或多个**ViewResoler**视图解析器，找到**ModelAndView**指定的视图

6)  视图负责将结果显示到客户端

---

## 二、注解

### 1、RequestMapping

- 使用 `@RequestMapping` 注解为控制器指定可以处理那些 URL 请求。

- 在控制器的类及方法上都可以标注

- `DispatherServlet` 截获请求后，就通过控制器上 `@RequestMapping` 提供的映射信息确定请求所对应的处理方法。

- `@RequestMapping` 的其他属性：

  - name

  - value

  - path：

    ```java
    /** 支持ant风格的路径匹配
    	?	匹配一个字符
    	*	匹配任意字符，
    	**	匹配多层路径
    */
    
    /**
    	PahVariable，路径上可以有一个占位符
    */
    ```

    

  - method：限定请求方法，`GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS, TRACE;`

  - params：规定请求参数，可以限定必须携带某个参数、不允许携带某个参数等

    ```java
    //必须携带username参数
    @RequestMapping(value = "/hello", params = {"username"})
    //禁止携带username参数
    @RequestMapping(value = "/hello", params = {"!username"})
    //禁止username参数为123
    @RequestMapping(value = "/hello", params = {"username!=123"})
    ```

  - headers：规定请求头中的内容

    ```java
    RequestMapping(value = "/something", headers = "content-type=text/*")
    ```

  - consumes：只接受内容类型是哪种的请求，规定请求头中的 `Content-Type`

  - produces：告诉浏览器返回的内容类型是什么，给响应头中加上`Content-Type`

### 2、RequestParam

`@RequestParam` 的属性：

- value：执行要获取参数的key
- name
- required：是否必须携带，默认为 `true`
- defaultValue：默认值，没有携带默认为 `null`

### 3、RequestHeader

获取请求头中的内容

```java
@Controller
@RequestMapping("/hi")
public class HelloController {
    @RequestMapping("/hello")
    public String hello(@RequestHeader("User-Agent") String userAgent) {
        System.out.println(userAgent);
        return "hello";
    }
}
```

### 4、CookieValue

获取 `cookie` 的值

## 三、数据输出

Spring MVC 除去出入原生的request和session外还能怎么样把数据带到页面？

### 1、Map、Model、ModelMap

在方法中传入 `Map`、`Model` 或者 `ModelMap`

在这些参数中保存的所有数据都会放入请求域中。

> `Map`、`Model` 或者 `ModelMap` 最终都是用 `BindingAwareModelMap` 在工作。

![image-20201027195422544](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201027195422544.png)

```java
@RequestMapping("/hello01")
public String hello02(ModelMap modelMap) {
    System.out.println(modelMap.getClass());
    modelMap.addAttribute("username", "zhangsan");
    return "hello";
}
```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h1>Hello...</h1>
    <h1>${requestScope.username}</h1>
</body>
</html>
```

### 2、ModelAndView

方法的返回值可以变成 `ModelAndView` 类型，既包括视图信息，也包含模型数据。（数据是放在请求域中）

## 四、源码解析

### 1、请求处理的流程分析

首先，`DispatcherServlet` 的结构如下

![image-20201028104324561](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201028104324561.png)

#### 1.1 `doDispatch` 方法

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

    try {
        ModelAndView mv = null;
        Exception dispatchException = null;

        try {
            // 1.检查是否是文件上传请求
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);

            // Determine handler for the current request.
            // 2. 根据当前的请求地址找到处理器（控制器）
            mappedHandler = getHandler(processedRequest);
            if (mappedHandler == null) {
                noHandlerFound(processedRequest, response);
                return;
            }

            // Determine handler adapter for the current request.
            // 3. 决定控制器类中的哪个方法（适配器）能够处理请求
            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

            ...

                // Actually invoke the handler.
                // 4. 适配器来执行目标方法，
                // 将目标方法执行完成后的返回值作为视图名，设置并保存到 ModelAndView 中
                // 无论目标方法怎么写，在经过适配器执行后都会返回 ModelAndView
                mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

            if (asyncManager.isConcurrentHandlingStarted()) {
                return;
            }

            applyDefaultViewName(processedRequest, mv);
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        }
        ...
            // 5. 转发到目标页面
            // 根据方法执行后封装的 ModelAndView，转发到对应页面
            processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    ...
}
```

![image-20210302093750659](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210302093750659.png)

#### 1.2 `getHandler` 方法

> `this.handlerMappings`： `SpringMVC`的九大组件之一，保存了

```java
// 根据当前请求在 handlerMappings 中找到这个请求的映射信息，获取目标处理器类
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		if (this.handlerMappings != null) {
			for (HandlerMapping mapping : this.handlerMappings) {
				HandlerExecutionChain handler = mapping.getHandler(request);
				if (handler != null) {
					return handler;
				}
			}
		}
		return null;
	}
```

![image-20201028155212628](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201028155212628.png)



#### 1.3 `getHandlerAdapter` 方法

> `this.handlerAdapters`： `SpringMVC`的九大组件之一

```java
// 根据当前处理器类，找到当前类的 HandlerAdapter
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
		if (this.handlerAdapters != null) {
			for (HandlerAdapter adapter : this.handlerAdapters) {
				if (adapter.supports(handler)) {
					return adapter;
				}
			}
		}
		throw new ServletException(...);
	}
```

![image-20201028155512718](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201028155512718.png)

#### 1.4 SpringMVC 的九大组件

`DispatcherServlet` 中有9个引用类型的属性，

SpringMVC在工作的时候，关键位置都是由这些组件完成的。

共同点：九大组件全部都是接口

```java
/** MultipartResolver used by this servlet. */
/** 文件上传解析器 */
@Nullable
private MultipartResolver multipartResolver;

/** LocaleResolver used by this servlet. */
/** 区域信息解析器，和国际化有关 */
@Nullable
private LocaleResolver localeResolver;

/** ThemeResolver used by this servlet. */
/** 主题解析器 */
@Nullable
private ThemeResolver themeResolver;

/** List of HandlerMappings used by this servlet. */
/** Handler映射信息 */
@Nullable
private List<HandlerMapping> handlerMappings;

/** List of HandlerAdapters used by this servlet. */
/** Handler的适配器 */
@Nullable
private List<HandlerAdapter> handlerAdapters;

/** List of HandlerExceptionResolvers used by this servlet. */
/** SpringMVC强大的异常解析功能，异常解析器 */
@Nullable
private List<HandlerExceptionResolver> handlerExceptionResolvers;

/** RequestToViewNameTranslator used by this servlet. */
@Nullable
private RequestToViewNameTranslator viewNameTranslator;

/** FlashMapManager used by this servlet. */
/** SpringMVC 中允许重定向携带数据的功能 */
@Nullable
private FlashMapManager flashMapManager;

/** List of ViewResolvers used by this servlet. */
/** 视图解析器 */
@Nullable
private List<ViewResolver> viewResolvers;
```

> 九大组件的初始化

![image-20201028162425468](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201028162425468.png)

> `handlerMappings` 的初始化

```java
/**
	 * Initialize the HandlerMappings used by this class.
	 * <p>If no HandlerMapping beans are defined in the BeanFactory for this namespace,
	 * we default to BeanNameUrlHandlerMapping.
	 */
private void initHandlerMappings(ApplicationContext context) {
    this.handlerMappings = null;

    if (this.detectAllHandlerMappings) {
        // Find all HandlerMappings in the ApplicationContext, including ancestor contexts.
        // 去容器中找这个组件（有些是按类型找，有些是按bean的id找）
        Map<String, HandlerMapping> matchingBeans =
            BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
        if (!matchingBeans.isEmpty()) {
            this.handlerMappings = new ArrayList<>(matchingBeans.values());
            // We keep HandlerMappings in sorted order.
            AnnotationAwareOrderComparator.sort(this.handlerMappings);
        }
    }
    ...

    // Ensure we have at least one HandlerMapping, by registering
    // a default HandlerMapping if no other mappings are found.
    if (this.handlerMappings == null) {
        // 如果没有找到任何的handlerMappings，就是用默认配置
        this.handlerMappings = getDefaultStrategies(context, HandlerMapping.class);
        ...
    }
}
```

#### 1.5 HandlerAdapter#handle

```java
//Actually invoke the handler.
```

#### 1.6 `processDispatchResult` 方法

1. 任何方法最终都会包装成 `ModelAndView` 对象

```java
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
			@Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
			@Nullable Exception exception) throws Exception {

		boolean errorView = false;

		if (exception != null) {
			...
		}

		// Did the handler return a view to render?
		if (mv != null && !mv.wasCleared()) {
            // 渲染页面
			render(mv, request, response);
			if (errorView) {
				WebUtils.clearErrorRequestAttributes(request);
			}
		}
    
    	...
	}
```

> `render()` 包括两步：
>
> 1. 获取视图对象 
> 2. 调用 view 的 `render` 方法渲染

```java
/* DispatcherServlet.java */
protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
		...

		View view;
		String viewName = mv.getViewName();
		if (viewName != null) {
			// 根据视图名 获取 View 视图对象
			view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
			if (view == null) {
				...
			}
		}
		...

		try {
			...
			view.render(mv.getModelInternal(), request, response);
		}
	}
```

##### 1.6.1 获取视图对象

> 根据视图名 获取 View 视图对象 === `resolveViewName(viewName, mv.getModelInternal(), locale, request);`。
>
> 视图解析器得到View对象的流程：所有配置的视图解析器都来尝试根据视图名得到View对象，如果得到就返回，得不到就换下一个视图解析器。
>
> `this.viewResolvers`： `SpringMVC`的九大组件之一

```java
protected View resolveViewName(String viewName, @Nullable Map<String, Object> model,
                               Locale locale, HttpServletRequest request) throws Exception {
    if (this.viewResolvers != null) {
        for (ViewResolver viewResolver : this.viewResolvers) {
            // viewResolver视图解析器根据方法的返回值，得到一个 View 对象
            View view = viewResolver.resolveViewName(viewName, locale);
            if (view != null) {
                return view;
            }
        }
    }
    return null;
}
```

![](https://gitee.com/yanjundong97/picBed/raw/master/images/20201031111625.png)

> 视图解析器获取 View 对象的方法：先从缓存中获取 视图对象，如果没有就创建

```java
public View resolveViewName(String viewName, Locale locale) throws Exception {
    ...
    else {
        // 根据视图名先从缓存中获取 视图对象，如果没有就创建
        Object cacheKey = getCacheKey(viewName, locale);
        View view = this.viewAccessCache.get(cacheKey);
        if (view == null) {
            synchronized (this.viewCreationCache) {
                view = this.viewCreationCache.get(cacheKey);
                if (view == null) {
                    // Ask the subclass to create the View object.
                    view = createView(viewName, locale);
                    ...
                }
            }
        }
        ...
        return (view != UNRESOLVED_VIEW ? view : null);
    }
}
```

> 创建 View 对象，如果是以 `redirect:` 前缀开始，就创建一个 `RedirectView` 对象；如果是以 `forward:` 前缀开始，就创建一个 `InternalResourceView` 对象；如果都不满足，就使用父类创建一个默认的View。

```java
@Override
protected View createView(String viewName, Locale locale) throws Exception {
    // If this resolver is not supposed to handle the given view,
    // return null to pass on to the next resolver in the chain.
    if (!canHandle(viewName, locale)) {
        return null;
    }

    // Check for special "redirect:" prefix.
    if (viewName.startsWith(REDIRECT_URL_PREFIX)) {
        String redirectUrl = viewName.substring(REDIRECT_URL_PREFIX.length());
        RedirectView view = new RedirectView(redirectUrl,
                                             isRedirectContextRelative(), isRedirectHttp10Compatible());
        String[] hosts = getRedirectHosts();
        if (hosts != null) {
            view.setHosts(hosts);
        }
        return applyLifecycleMethods(REDIRECT_URL_PREFIX, view);
    }

    // Check for special "forward:" prefix.
    if (viewName.startsWith(FORWARD_URL_PREFIX)) {
        String forwardUrl = viewName.substring(FORWARD_URL_PREFIX.length());
        InternalResourceView view = new InternalResourceView(forwardUrl);
        return applyLifecycleMethods(FORWARD_URL_PREFIX, view);
    }

    // Else fall back to superclass implementation: calling loadView.
    return super.createView(viewName, locale);
}
```

##### 1.6.2 视图渲染

> 视图创建完成后，会调用 view 的 `render` 方法

```java
/* AbstractView.java */
@Override
public void render(@Nullable Map<String, ?> model, HttpServletRequest request,
                   HttpServletResponse response) throws Exception {
    Map<String, Object> mergedModel = createMergedOutputModel(model, request, response);
    prepareResponse(request, response);
    // 渲染要给页面输出的所有数据
    renderMergedOutputModel(mergedModel, getRequestToExpose(request), response);
}
```

> `InternalResourceView.java`

```java
/* InternalResourceView.java */
@Override
protected void renderMergedOutputModel(
    Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {

    // Expose the model object as request attributes.
    // 将模型中的数据放在请求域中
    exposeModelAsRequestAttributes(model, request);

    // Expose helpers as request attributes, if any.
    exposeHelpers(request);

    // Determine the path for the request dispatcher.
    String dispatcherPath = prepareForRendering(request, response);

    // Obtain a RequestDispatcher for the target resource (typically a JSP).
    RequestDispatcher rd = getRequestDispatcher(request, dispatcherPath);
    ...

    else {
        // Note: The forwarded resource is supposed to determine the content type itself.
        ...
        // 转发到页面
        rd.forward(request, response);
    }
}
```

##### 1.6.3 总结

`视图解析器`只是为了得到`视图View对象`，试图对象才能转正的转发（将模型数据全部放入请求域中）或者重定向到页面。

视图对象才能真正的渲染视图。

#### 1.7 总结

- 请求处理方法执行完成后，最终返回一个 `ModelAndView` 对象。无论返回类型是`String、View或者ModelMap`，`SpringMVC`在内部都会把他们装配成一个 `ModelAndView` 对象，它包含了逻辑名和模型对象的视图。
- `SpringMVC` 借助视图解析器（`ViewResolver`）得到最终的视图对象，最终的视图可以是`JSP`、也可以是`Excel`等各种表现形式的视图。
- 对于最终究竟采取何种视图对象对模型数据进行渲染，处理器并不关心，处理器的==工作重点聚焦在生产模型数据的工作上==，从而实现 MVC 的充分解耦。

### 2、View接口

![](https://gitee.com/yanjundong97/picBed/raw/master/images/20201031162455.png)

- `视图（View）`的作用是渲染模型数据，将模型里的数据以某种形式呈现给客户。

- ==视图对象由`视图解析器`负责实例化==。由于视图是**无状态**的，所以他们不会有**线程安全**的问题

- 常用的视图实现类

  ![](https://gitee.com/yanjundong97/picBed/raw/master/images/20201031162254.png)

### 3、ViewResolver接口

![](https://gitee.com/yanjundong97/picBed/raw/master/images/20201031162747.png)

- •视图解析器的作用比较单一：==将逻辑视图解析为一个具体的视图对象==。

- `SpringMVC` 为逻辑视图名的解析提供了不同的策略，可以在 `Spring WEB` 上下文中**配置一种或多种解析策略**，并指定他们之间的先后顺序。每一种映射策略对应一个具体的视图解析器实现类。

- 可以选择一种视图解析器或混用多种视图解析器

- `SpringMVC` 会按视图解析器顺序的优先顺序对逻辑视图名进行解析，直到解析成功并返回视图对象，否则将抛出 `ServletException` 异常

- 常用的视图解析器实现类

  ![](https://gitee.com/yanjundong97/picBed/raw/master/images/20201031162927.png)



## 五、数据转换&数据格式化&数据校验

```java
/** ModelAttributeMethodProcessor.java */
public final Object resolveArgument(...) throws Exception {
    ...

    if (bindingResult == null) {
        WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);
        if (binder.getTarget() != null) {
            if (!mavContainer.isBindingDisabled(name)) {
                this.bindRequestParameters(binder, webRequest);
            }

            this.validateIfApplicable(binder, parameter);
            if (binder.getBindingResult().hasErrors() && 
                this.isBindExceptionRequired(binder, parameter)) {
                throw new BindException(binder.getBindingResult());
            }
        }

        ...

        bindingResult = binder.getBindingResult();
    }

    Map<String, Object> bindingResultModel = bindingResult.getModel();
    mavContainer.removeAttributes(bindingResultModel);
    mavContainer.addAllAttributes(bindingResultModel);
    return attribute;
}
```

方法调用栈：

![image-20201101144838278](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201101144838278.png)

`WebDataBinder` ：数据绑定器负责数据绑定工作。数据绑定期间产生的类型转换、格式化、数据校验等问题。

![image-20201101144251356](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201101144251356.png)

### 1、数据校验

- `JSR 303` 是 Java 为 Bean 数据合法性校验提供的标准框架，它已经包含在 JavaEE

- 通过在 Bean属性上标注类似于 @NotNull、@Max 等标准的注解指定校验规则，并通过标准的验证接口对 Bean 进行验证

  ![image-20201101154829958](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201101154829958.png)

- `Hibernate Validator` 是 JSR 303 的一个参考实现，除支持所有标准的校验注解外，它还支持以下的扩展注解

  ![image-20201101155259582](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201101155259582.png)

## 六、支持前后端分离

### 1、Jackson

- `Jackson`导包

![image-20201101162527158](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201101162527158.png)

- `Jackson` 的注解

![image-20201101162442802](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201101162442802.png)

### 2、`HttpMessageConverter`

`HttpMessageConverter<T>` 是一个接口，==负责将请求信息转换为一个对象（类型为T）==，==将对象（类型为T）输出位响应信息==。

`HttpMessageConverter<T>`接口定义的方法：

- `boolean canRead(Class<?> var1, @Nullable MediaType var2);`：指定转换器可以读取的对象类型，即转换器是否可将请求信息转换为 clazz 类型的对象，同时指定支持 MIME 类型(text/html,applaiction/json等)

- `boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);`：指定转换器是否可将 clazz 类型的对象写到响应流中，响应流支持的媒体类型在MediaType 中定义。
- `List<MediaType> getSupportedMediaTypes();`：该转换器支持的媒体类型。
- `T read(Class<? extends T> clazz, HttpInputMessage inputMessage);`：将请求信息流转换为 T 类型的对象。
- `void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage);`：将T类型的对象写到响应流中，同时指定相应的媒体类型为 contentType。

![image-20201101175730048](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201101175730048.png)

![image-20201101180214519](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201101180214519.png)



## 七、拦截器

### 1、实现一个拦截器

```java
public class MyFirstInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("MyFirstInterceptor#preHandle...");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("MyFirstInterceptor#postHandle...");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("MyFirstInterceptor#afterCompletion...");
    }
}
```

```xml
<mvc:interceptors>
    <!-- 默认拦截所有的请求 -->
    <bean class="com.coder.MyFirstInterceptor"></bean>
    <!-- 拦截特定的请求 -->
    <!--<mvc:interceptor>
            <mvc:mapping path="/hello"/>
            <bean class="com.coder.MyFirstInterceptor"></bean>
        </mvc:interceptor>-->
</mvc:interceptors>
```



执行流程：

```tex
MyFirstInterceptor#preHandle...
控制器执行...
MyFirstInterceptor#postHandle...
hello.jsp
MyFirstInterceptor#afterCompletion...
```

### 2、多个拦截器

```java
public class MySecondInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("MySecondInterceptor#preHandle...");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("MySecondInterceptor#postHandle...");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("MySecondInterceptor#afterCompletion...");
    }
}
```

```tex
MyFirstInterceptor#preHandle...
MySecondInterceptor#preHandle...
控制器执行...
MySecondInterceptor#postHandle...
MyFirstInterceptor#postHandle...
hello.jsp
MySecondInterceptor#afterCompletion...
MyFirstInterceptor#afterCompletion...
```

> 拦截器的`preHandle`：是按顺序执行
>
> 拦截器的`postHandle`：是按逆序执行
>
> 拦截器的`afterCompletion`：是按逆序执行
>
> 已经放行的拦截器的`afterCompletion` 总会执行

异常情况：

- 不放行，哪一块不放行从此以后的都没有

- `MySecondInterceptor` 不放行，但是它前面已经放行的拦截器的`afterCompletion` 总会执行。

  ```tex
  MyFirstInterceptor#preHandle...
  MySecondInterceptor#preHandle...
  MyFirstInterceptor#afterCompletion...
  ```

  

### 3、执行流程分析

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    ...

    try {
        ModelAndView mv = null;
        Exception dispatchException = null;

        try {
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);

            // 拿到方法的执行链，包括拦截器
            mappedHandler = getHandler(processedRequest);
            ...

            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

            ...

        //拦截器preHandle执行位置；有一个拦截器返回false，目标方法以后都不会执行，直接跳到afterCompletion
            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }

            
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

            if (asyncManager.isConcurrentHandlingStarted()) {
                return;
            }

            applyDefaultViewName(processedRequest, mv);
            // 目标方法只要正常就会执行postHandle
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        }
        catch (Exception ex) {
            dispatchException = ex;
        }
        catch (Throwable err) {
            dispatchException = new NestedServletException("Handler dispatch failed", err);
        }
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    catch (Exception ex) {
        triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
    }
    catch (Throwable err) {
        triggerAfterCompletion(processedRequest, response, mappedHandler,
                               new NestedServletException("Handler processing failed", err));
    }
    finally {
        ...
    }
}
```



> `applyPreHandle` `applyPostHandle`  `triggerAfterCompletion`

```java
/** HandlerExecutionChain.java */
/**
	 * Apply preHandle methods of registered interceptors.
	 * @return {@code true} if the execution chain should proceed with the
	 * next interceptor or the handler itself. Else, DispatcherServlet assumes
	 * that this interceptor has already dealt with the response itself.
	 */
boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HandlerInterceptor[] interceptors = getInterceptors();
    if (!ObjectUtils.isEmpty(interceptors)) {
        // preHandle 顺序执行
        for (int i = 0; i < interceptors.length; i++) {
            HandlerInterceptor interceptor = interceptors[i];
            if (!interceptor.preHandle(request, response, this.handler)) {
                // 存在不放行的拦截器，直接跳转到 afterCompletion
                triggerAfterCompletion(request, response, null);
                return false;
            }
            // 并且会记录最后一次放行的拦截器索引，在执行 afterCompletion 时需要
            this.interceptorIndex = i;
        }
    }
    return true;
}


/**
	 * Apply postHandle methods of registered interceptors.
	 */
void applyPostHandle(HttpServletRequest request, HttpServletResponse response, @Nullable ModelAndView mv)
    throws Exception {

    HandlerInterceptor[] interceptors = getInterceptors();
    if (!ObjectUtils.isEmpty(interceptors)) {
        // postHandle 逆序执行
        for (int i = interceptors.length - 1; i >= 0; i--) {
            HandlerInterceptor interceptor = interceptors[i];
            interceptor.postHandle(request, response, this.handler, mv);
        }
    }
}

/**
	 * Trigger afterCompletion callbacks on the mapped HandlerInterceptors.
	 * Will just invoke afterCompletion for all interceptors whose preHandle invocation
	 * has successfully completed and returned true.
	 */
void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, @Nullable Exception ex)
    throws Exception {

    HandlerInterceptor[] interceptors = getInterceptors();
    if (!ObjectUtils.isEmpty(interceptors)) {
        // 有记录最后一个放行拦截器的索引，从它开始倒序执行之前所有拦截器的afterCompletion
        for (int i = this.interceptorIndex; i >= 0; i--) {
            HandlerInterceptor interceptor = interceptors[i];
            try {
                interceptor.afterCompletion(request, response, this.handler, ex);
            }
            catch (Throwable ex2) {
                logger.error("HandlerInterceptor.afterCompletion threw exception", ex2);
            }
        }
    }
}
```

## 八、异常处理

`Spring MVC` 通过 `HandlerExceptionResolver` 处理程序的异常，包括Handler映射、数据绑定以及目标方法执行时发生的异常。

### 1、流程分析

> `handlerExceptionResolvers` 的初始化

```java
private void initHandlerExceptionResolvers(ApplicationContext context) {
    this.handlerExceptionResolvers = null;

    if (this.detectAllHandlerExceptionResolvers) {
        // 获取所有的 handlerExceptionResolver
        Map<String, HandlerExceptionResolver> matchingBeans = BeanFactoryUtils
            .beansOfTypeIncludingAncestors(context, HandlerExceptionResolver.class, true, false);
        if (!matchingBeans.isEmpty()) {
            this.handlerExceptionResolvers = new ArrayList<>(matchingBeans.values());
            // We keep HandlerExceptionResolvers in sorted order.
            AnnotationAwareOrderComparator.sort(this.handlerExceptionResolvers);
        }
    }
    
    if (this.handlerExceptionResolvers == null) {
        // 没有获取到，获取默认的 handlerExceptionResolvers
        this.handlerExceptionResolvers = getDefaultStrategies(context, HandlerExceptionResolver.class);
        
    }
}
```

> 默认的 `handlerExceptionResolvers `

```pro
# 处理标注了@ExceptionHandler的异常
org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver，
# 处理标注了@ResponseStatus的异常
org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,
# 处理SpringMVC自带的异常
org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver
```

#### 1.1 异常处理

> `DispatcherServlet#doDispatch` ，如果目标方法执行有异常，就会初始化 `dispatchException` ，并作为参数传入 `DispatcherServlet#processDispatchResult` 进行处理

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) {
    try {
        ...
        try {
            ...

            // Actually invoke the handler.
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
            ...
        }
        catch (Exception ex) {
            dispatchException = ex;
        }
        
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
}
```



> `DispatcherServlet#processHandlerException` ，使用默认的 `handlerExceptionResolvers` 去解析，如果都不能处理都直接抛出去

```java
/** DispatcherServlet.java */
@Nullable
protected ModelAndView processHandlerException(HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex) {

    ModelAndView exMv = null;
    if (this.handlerExceptionResolvers != null) {
        // 使用默认的异常解析器去解析，如果能够处理就跳出循环
        for (HandlerExceptionResolver resolver : this.handlerExceptionResolvers) {
            exMv = resolver.resolveException(request, response, handler, ex);
            if (exMv != null) {
                break;
            }
        }
    }
    if (exMv != null) {
        ...
        // 返回错误页面
        return exMv;
    }

    throw ex;
}
```

![image-20201101224551714](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201101224551714.png)

### 2、`@ExceptionHandler`

```java
/**
 * 1. 集中处理所有异常的类加入ioc容器中
 * 2. @ControllerAdvice 专门处理异常的类
 */
@ControllerAdvice
public class MyExceptionHandler {

    @ExceptionHandler(value = Exception.class)
    public String nullExceptionHandler(Exception e) {
        return "error";
    }

}
```

> `@ExceptionHandler` 注解定义的方法**优先级问题**：例如发生的是`NullPointerException`，但是声明的异常有 `RuntimeException` 和 `Exception`，此候会根据异常的最近继承关系找到继承深度最浅的那个 `@ExceptionHandler` 。

### 3、`@ResponseStatus`

用在自定义异常上

```java
@ResponseStatus(reason = "用户名没有找到", code = HttpStatus.LOCKED)
public class UserNameNotFoundException extends RuntimeException {
}
```

![image-20201101225013526](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201101225013526.png)

### 4、`DefaultHandlerExceptionResolver`

对一些特殊的异常进行处理，比如`NoSuchRequestHandlingMethodException`、`HttpRequestMethodNotSupportedException`、`HttpMediaTypeNotSupportedException`、`HttpMediaTypeNotAcceptableException`等


















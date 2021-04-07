# 一、SpringApplication

`SpringApplication` 类提供了一个方便的方法来启动 `Spring` 项目。

```java
public static void main(String[] args) {
    SpringApplication.run(MySpringConfiguration.class, args);
}
```

在应用程序启动时，可以看下类似下面的输出内容：

```tex
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.1.RELEASE)

2020-03-03 13:57:35.594  INFO 5342 --- [           main] c.s.s.Springboot01HelloworldApplication  : Starting Springboot01HelloworldApplication on yanjundong with PID 5342 (/Users/yanjundong/IdeaProjects/springboot-01-helloworld/target/classes started by yanjundong in /Users/yanjundong/IdeaProjects/springboot-01-helloworld)

```

默认情况下，输入 `INFO` 级别日志消息，包括启动相关的信息，例如启动应用程序的用户。

如果需要 ==调整日志的输出级别== ，可以另外设置。

## 1、启动失败

## 2、延迟初始化（Lazy Initialization）

 `SpringApplication`允许延迟初始化应用程序。启用惰性初始化后，将根据需要创建bean，而不是在应用程序启动时就创建bean。因此，启动延迟初始化可以减少应用程序启动花费的时间。在web应用中，启动延迟初始化将导致许多web相关的bean直到接收到了 `HTTP请求` 才会被初始化。

延迟初始化的缺点：

- 不能够及时发现应用的问题。如果一个配置错误的bean被延迟初始化，那这个错误就只能在启动之后才会发现。
- 必须要确保JVM有足够的内存来容纳所有的bean，而不仅仅是启动时初始化的bean。

基于以上原因，默认情况下，是没有启动延迟初始化的。

启动延迟初始化的方式：

- 使用 `SpringApplicationBuilder` 中的 `lazyInitialization` 方法

- 使用 `SpringApplication` 中的 `setLazyInitialization` 方法

- 使用 `spring.main.lazy-initialization` 属性配置

  ```properties
  spring.main.lazy-initialization=true
  ```

> 如果应用程序启动了延迟初始化，但是想要对某些bean禁用延迟初始化，可以使用 `@Lazy(false)` 将延迟属性设置为false



## 3、自定义Banner

将 `banner.txt` 文件添加到类路径（maven项目中的 `resources` 下）或者也可以将 `spring.banner.location` 属性设置为此类文件的位置来更改启动时打印的横幅。

如果文件的编码方式不是 `UTF-8` ，则可以设置  `spring.banner.charset`。

除了文本文件，还可以添加`banner.gif`，`banner.jpg`或`banner.png`图像文件，用法和 `banner.txt` 一样。

在`banner.txt`文件内部，可以使用以下任意占位符：

| 变量                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ${application.version}                                       | 用来获取`MANIFEST.MF`文件中的版本号。例如`1.0`               |
| ${application.formatted-version}                             | 格式化后的`${application.version}`版本信息。例如： `v1.0`    |
| ${spring-boot.version}                                       | 当前使用Spring Boot的版本。例如 `2.2.4.RELEASE`              |
| ${spring-boot.formatted-version}                             | 格式化后的`${spring-boot.version}`版本信息。例如`v2.2.4.RELEASE` |
| `${Ansi.NAME}` (or `${AnsiColor.NAME}`, `${AnsiBackground.NAME}`, `${AnsiStyle.NAME}`) |                                                              |
| ${application.title}                                         | 用来获取`MANIFEST.MF`文件中的应用程序的标题。例如`MyApp`     |

可以通过配置 `spring.main.banner-mode`属性的值来确定banner的输出位置：

- `console`—— System.out（console）打印
- log——配置的logger
- off——不制作

默认打印的banner是作为一个单独的bean注册在 `SpringBootBanner` 中。

Banner字符可以通过以下网站生成：

- http://patorjk.com/software/taag
- http://www.network-science.de/ascii/

## 4、自定义 SpringApplication

如果不满足`SpringApplication`的默认设置，则可以创建一个本地实例并对其进行自定义。例如，要关闭横幅，您可以编写：

```java
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(MySpringConfiguration.class);
    app.setBannerMode(Banner.Mode.OFF);
    app.run(args);
}
```

> 传递给构造函数的参数 `MySpringConfiguration.class` 是Spring bean 的配置源。在大多数情况下，这些是注解了 `@Configuration`  的类。

也可以使用`application.properties`文件来配置。



# 二、外部配置

Spring Boot中可以使用外部配置，以便在不同的环境中使用相同的代码。你可以使用 `properties` 文件、`YAML` 文件、`环境变量` 和`命令行参数` 来配置。

## 1、配置随机值

`RandomValuePropertySource` 在注入随机值时是十分有用的。例如进入隐私的或者测试的例子。它可以产生整数、longs、uuid或字符串，如下所示：

```properties
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}
```

`random.int*` 语法可以带有参数。

- random.int
- random.int(value)
- random.int(value, max) ——如果有`max`，`value`提供的就是最小值。`value` 和 `max` 都是整数。

## 2、命令行配置属性

所有的属性配置都可以在命令行上进行指定。例如：

```sh
java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --server.port=8087  --server.context-path=/abc
```

默认情况下，`SpringApplication`将所有命令行选项参数转换为a `property`并将其添加到Spring `Environment`。

如果不希望将命令行属性添加到`Environment`，可以使用`SpringApplication.setAddCommandLineProperties(false)`来禁用。

## 3、Application Property 文件

`SpringApplication`从以下位置的`application.properties`文件中加载属性，并将其添加到Spring `Environment`：

1. 当前目录的一个 `/config` 子目录下（`file:./config/`）；
2. 当前目录（`file:./`）；
3. 类路径的 `/config` 包（`classpath:/config/`）；
4. 类路径下（`classpath:/`）

优先级由高到底，高优先级的配置会覆盖低优先级的配置；

> 可以使用YAML（.yml）来代替 `.properties`



如果不喜欢`application.properties`作为配置文件的名称，可以通过指定 `spring.config.name` 来使用另一个文件名。另外，还可以配置 `spring.config.location` 来改变默认的配置文件位置。

```sh
#指定其他文件名
$ java -jar myproject.jar --spring.config.name = myproject
```

项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置。

如果要指定配置文件和默认加载的这些配置文件共同起作用形成互补配置，需要使用`spring.config.additional-lacation`；

```sh
#指定配置文件加载位置，如果有多个，用逗号隔开
$ java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --spring.config.location=G:/application.properties
```

**配置文件是以相反的顺序搜索**。默认情况下，`configured locations` 是 `classpath:/,classpath:/config/,file:./,file:./config/`。但搜索顺序的结果正好相反。

当通过 `spring.config.lacation` 自定义配置位置后，它们就会**取代**默认位置。并且如果`spring.config.location`配置为`classpath:/custom-config/,file:./custom-config/`，则搜索顺序将变为：

1. `file:./custom-config/`
2. `classpath:custom-config/`

当使用`spring.config.additional-location`配置时，除了默认位置外，还会使用配置的值。

> 1. 可以在一个配置文件中指定默认值，然后在另一配置文件中选择性的覆盖这些值。
> 2. 如果你使用 environment变量而不是 system 属性，大多数操作系统都不允许使用`.`分割的键名，这是可以使用`_` 代替。（`SPRING_CONFIG_NAME`代替`spring.config.name`）。

## 4、profile

Profile可以为不同环境提供不同配置，可以通过激活、指定参数等方式快速切换环境。

在编写配置文件时，文件名可以是 `application-{profile}.properties`。默认使用`application-default.properties`（即`application.properties`）加载属性。

 `application-{profile}.properties`文件能够从`application.properties` 的位置加载。

不论`application-{profile}.properties` 文件是在jar包外还是内部，他都可以覆盖`application.properties` 

用法：

指定两种环境，分别为dev 和 prod：

```sh
#application-dev.properties
server.port=8081
```

```sh
#application-prod.properties
server.port=8082
```

激活profile有三种方法：

1. 配置文件中指定

   ```properties
   #application.properties
   spring.profiles.active=dev	#启动后端口号为8081
   ```

2. 命令行指定

   ```sh
   java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev；
   ```

   也可以在idea启动前，配置传入参数：

   ![](https://gitee.com/yanjundong97/picBed/raw/master/images/WX20200303-164921.png)

3. 虚拟机指定

   如上图

## 5、加密属性值

Spring Boot不提供任何内置的对属性值加密的支持。

## 6、YAML的使用

Spring Framework提供了两个方便的类来加载YAML文件。

- `YamlPropertiesFactoryBean` 将YAML 加载为 `Properties`
- `YamlMapFactoryBean` 将YAML 加载为 `Map`

### 6.1 YAML语法

```yaml
environments:
    dev:
        url: https://dev.example.com
        name: Developer Setup
    prod:
        url: https://another.example.com
        name: My Cool App
```

以上的YAML文档将会转化为以下properties：

```properties
environments.dev.url=https://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=https://another.example.com
environments.prod.name=My Cool App
```

**List表示：**

```yaml
my:
   servers:
       - dev.example.com
       - another.example.com
```

以上的YAML文档将会转化为以下properties：

```properties
my.servers[0]=dev.example.com
my.servers[1]=another.example.com
```

### 6.2 多文档块方式

```yaml
server:
  port: 8081
#spring:
#  profiles:
#    active: dev
---
server:
  port: 8082
spring:
  profiles: dev
---
server:
  port: 8083
spring:
  profiles: prod
```

在上面的例子中，如果`dev`配置文件处于活动状态，则`server.port`属性为`8082`。如果`prod`配置文件处于活动状态，则`server.port`属性为`8083`。上面的文档中`spring.profiles.active` 注释了，因此`server.port`属性为`8081`。

### 6.3 YAML的缺点

不能通过`@PropertySource` 来加载YAML配置文件，所以如果想要通过这种方式加载值，只能使用`properties` 文件。

```properties
#person.properties
person.name: zhangsan
person.age: 10
person.address: 陕西省
person.birthday: 2017/10/20
```

```java
@Component
@PropertySource(value = {"classpath:person.properties"})
@ConfigurationProperties(prefix = "person")
public class Person {

    private String name;
    private int age;
    private String address;
    private Date birthday;
  	//...省略getter和setter方法
}
```

在profile中使用YAML的多文档块方式可以会出现问题。

官方建议不要混合使用 `YAML格式的profile文档` 和`YAML的多文档块` ，坚持只使用其中一个。

## 7、配置属性的类型安全

### 7.1 配置文件注入JavaBean

```yaml
#application.yml
person:
    lastName: hello
    age: 18
    boss: false
    birth: 2017/12/12
    maps: {k1: v1,k2: 12}
    lists:
      - lisi
      - zhaoliu
    dog:
      name: 小狗
      age: 12
```

```java
/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 * @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
 *      prefix = "person"：配置文件中person下面的所有属性进行一一映射
 *
 * 使用@Component将该类称为容器中的组件
 * 只有这个组件是容器中的组件，才能使用容器提供的@ConfigurationProperties功能；
 *
 */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
  	//...省略getter和setter方法
}
```

> 可以通过 `properties文件` 、 `YAML文件` 、 `外部环境变量`等配置。
>
> 通常情况下，`getters/setters` 是强制要求的，因为值的绑定是通过标准的 Java Beans 属性描述符实现的。在下列情况中，`setter` 才可以忽略：
>
> 

### 7.2 构造函数绑定

==测试出现问题==



### 7.3 @ConfigurationProperties



### 7.4 松散绑定

SpringBoot使用松散的规则将 `properties` 绑定到 `@ConfigurationProperties` 注解的beans上。

常见的例子包括

- `user-name` 绑定到 `userName`
- `PORT` 绑定到 `port`

现有以下 `@ConfigurationProperties` 注解的类

```java
@Component
@ConfigurationProperties(prefix="person")
public class person {

    private String firstName;

    public String getFirstName() {
        return this.firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

}
```

使用上面的的代码，下面的这些 `properties` 名称都能够使用：

| Properties        | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| person.first-name | 短横线命名，建议在`.properties`和`.yml`文件中使用            |
| person.firstName  | 标准驼峰式命名                                               |
| person.first_name | 下划线表示法，是在`.properties`和`.yml`文件中使用的另一种格式。 |
| PERSON_FORSTNAME  | 大写的格式，通常使用于系统环境变量                           |

> `prefix` 的值必须使用 `短横线命名法` （小写字母，以`-`分隔开，例如 `acme.my-project.person`）



#### 不同属性的松散绑定规则

| Property Source | 简单                                               | List                                                         |
| --------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| Properties文件  | 短横线命名，驼峰式命名 或 下划线表示法             | 使用[ ]或逗号分隔的标准列表语法                              |
| YAML文件        | 短横线命名，驼峰式命名 或 下划线表示法             | YAML列表语法或者逗号分隔的值                                 |
| 环境变量        | 以`_`作为定界符的大写格式，`_`不应在属性名称中使用 | 下划线括起来的数值，例如 `MY_ACME_1_OTHER = my.acme[1].other` |
| 系统属性        | 短横线命名，驼峰式命名 或 下划线表示法             | 使用[ ]或逗号分隔的标准列表语法                              |

### 7.5 @ConfigurationProperties数据校验

如果 `@ConfigurationProperties` 标注的类被 `@Validated` 注解，SpringBoot就会对该类的数据进行校验。

你能够直接使用 `JSR-303` 数据校验。

```java
@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {
    @Email	//必须是邮箱
    private String email;
  	@NotNull	//不能为null
    private InetAddress remoteAddress;
}
```

为了确保嵌套属性的验证，相关的字段必须使用 `@Valid` 注解，示例：

```java
/*Dog.java*/
public class Dog {

    private Long id;
    @Email
    private String name;
    private String type;
  	//...省略getter和setter方法
}
```

```java
/*Person.java*/
@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {
    @NotNull
    private String name;
    private int age;
    private String address;
    private Date birthday;

    @Valid
    private Dog dog = new Dog();
  
  	//...省略getter和setter方法
}
```

你可以通过创建一个名为`configurationPropertiesValidator` 的bean自定义Spring 的`Validator` ，该`@Bean` 的方法应该声明为`static` 。



### 7.6 @ConfigurationProperties与@Value

|                | @ConfigurationProperties | @Value     |
| -------------- | ------------------------ | ---------- |
| 功能           | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定       | 支持                     | 不支持     |
| ==SpEL==       | 不支持                   | 支持       |
| JSR303数据校验 | 支持                     | 不支持     |
| 复杂类型封装   | 支持                     | 不支持     |

如果我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用`@Value`；

如果我们专门编写了一个javaBean来和配置文件进行映射，我们就直接使用`@ConfigurationProperties`；



最后，虽然你能够在`@Value` 写一个`SPEL` 表达式，但从`application property ` 文件中获取的属性表达式不能处理。

```java
@Value("${person.Name}")	//可以获取到properties中的值
private String name;
@Value("${person.age * 2}")	//可以获取到值，但不能处理，该处报错！！！
private int age;
```

# 三、Profiles

SpringBoot 提供了一个方法使某个配置文件只在特定的环境中加载。任何注解了 `@Component`, `@Configuration` 和 `@ConfigurationProperties` 的，都可以用 `@Profile` 限制它们的加载。如下例：

```java
@Configuration(proxyBeanMethods = false)
@Profile("production")
public class ProductionConfiguration {

    // ...

}
```

可以使用 `properties` 文件或者 `命令行` 的方式指定活动状态。例如：

```properties
spring.profiles.active=dev,hsqldb
```

```sh
--spring.profiles.active=dev,hsqldb
```

## 1、添加Profile配置文件

上面的 `二/4` 和 `二/6/6.2` 已经写了。

# 四、日志

日志框架包括两种：

- 日志门面——日志的一个抽象层
- 日志的实现

| 日志的抽象层                                                 | 日志的实现                                         |
| ------------------------------------------------------------ | -------------------------------------------------- |
| JCL（Jakarta  Commons Logging）    SLF4j（Simple  Logging Facade for Java）    jboss-logging | Log4j  、JUL（java.util.logging）  Log4j2  Logback |

在使用时应该从左边选取一个抽象层，右边选取一个实现框架。

SpringBoot使用 `Commons Logging` 作为内部日志记录，但保留了底层的日志实现。提供了对于`java util logging`、`Log4j2` 、`Logback`的默认配置。logger都预先配置为使用控制台输出，但可以选择使用文件输出。

默认情况下，使用`Logback` 进行日志记录。

## 1、日志格式

SpringBoot的默认日志输出类似于：

```
2020-03-04 13:52:20.301  INFO 7169 --- [           main] s.Springboot01HelloworldApplicationTests : No active profile set, falling back to default profiles: default
2020-03-04 13:52:21.097  INFO 7169 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-03-04 13:52:21.312  INFO 7169 --- [           main] s.Springboot01HelloworldApplicationTests : Started Springboot01HelloworldApplicationTests in 1.317 seconds (JVM running for 2.174)

```

输出以下项：

- 日期和时间：精度为毫秒，易于排序
- 日志级别： `ERROR`, `WARN`, `INFO`, `DEBUG`, or `TRACE`
- 进程ID
- `---` 标志日志信息的开始
- 线程名称：用方括号扩起来（在控制台输出时可以会被截断）
- 记录器名称：通常是类名（通常缩写）
- 日志消息

> Logback没有 `FATAL` 级别，归于了 `ERROR`。

## 2、控制台输出

默认情况下，日志会输出到控制台上。默认`ERROR` 、`WRAN` 和`INFO` 级别的日志会被记录。能够通过`--dubug`来启动应用程序开启“debug”模式。

```sh
$ java -jar myapp.jar --debug
```

> 也能够在 `application.properties` 中指定`debug=true`

开启“debug”模式后，一些核心的logger（内置的容器, Hibernate, 并且 Spring Boot）会被配置来输出更多的信息。启用 `debug` 模式不会将应用程序配置为以debug级别记录所有消息。

当然也可以通过同样的方式开启 `trace` 模式。

### 2.1 彩色输出

如果你的 `terminal` 支持 `ANSI` ，彩色输出能够提高可读性。可以通过设置  `spring.output.ansi.enabled`  为`支持的值` 来改变默认的自动检测(detect)。

通过使用 `%clr` 转化字符来配置彩色编码。如下示例：

```properties
%clr(%5p)
```

| Level   | Color  |
| :------ | :----- |
| `FATAL` | Red    |
| `ERROR` | Red    |
| `WARN`  | Yellow |
| `INFO`  | Green  |
| `DEBUG` | Green  |
| `TRACE` | Green  |

另外，你可以通过将颜色或样式作为转换选项来指定应该使用的颜色或样式。在下面的例子中，会使文本变成黄色：

```properties
%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}
```

支持的颜色和样式：

- `blue`
- `cyan`
- `faint`
- `green`
- `magenta`
- `red`
- `yellow`

## 3、文件输出

SpringBoot的日志默认只会输出在控制台中，如果想要记录日志到文件中，需要设置`logging.file.name` 和 `logging.file.path` 属性（可以在`application.properties`文件）。

下面的表格显示了  `logging.*` 如果配合使用：

| `logging.file.name` | `logging.file.path` | Example    | Description                                                  |
| :------------------ | :------------------ | :--------- | :----------------------------------------------------------- |
| *(无)*              | *(无)*              |            | 仅仅在控制台输出                                             |
| Specific file       | *(无)*              | `my.log`   | 写入指定的目录文件。名称可以是绝对位置或相对位置             |
| *(无)*              | Specific directory  | `/var/log` | 把`spring.log`文件写入 指定的目录。名称可以是绝对位置或相对位置 |

当日志文件达到10 MB时，就会循环保存（意味着1个文件的大小只能是10MB，达到后就会换一个文件存放），并且与控制台输出一样，默认情况下会记录 `ERROR/WRAN/INFO` 日志 。

可以通过 `logging.file.max-size` 属性改变文件大小限制。

除非 `logging.file.max-history` 已经被设置，否则默认情况下保存最近7天的循环日志文件。

日志档案的总大小能够通过 `logging.file.total-size-cap` 设置上限，当总大小超过该阀值，就会删除备份。

要在应用程序启动时强制清除日志存档，需要使用 `logging.file.clean-history-on-start` 属性。

> 每一个日志的实现框架都有自己的配置文件。不论使用哪个日志接口框架，**配置文件还是做成日志实现框架自己本身的配置文件；**

## 4、日志级别

日志的输出级别可以在Spring `Environment` 配置（例如：`application.properties` ，环境变量）。

配置方法：

- `logging.level.<logger-name>=<level>` ，其中的`level` 可选为：`TRACE` 、`DEBUG`、 `INFO`、 `WRAN`、 `ERROR`、 `FATAL`、 `OFF` 。

  ```properties
  #root的logger能够通过 `loggin.level.root` 配置
  logging.level.root=warn	
  logging.level.org.springframework.web=debug
  logging.level.org.hibernate=error
  logging.level.com.springboot=trace
  ```

- 环境变量配置：`LOGGING_LEVEL_ORG_SPRINGFRAMEWORK_WEB=DEBUG` 将设置`org.springframework.web` 的输出为`DEBUG` 级别。

## 5、日志分组

能够把相关的logger放到一个组统一管理。

例如，定一个“tomcat”组：

```properties
logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat
```

统一管理“tomcat”组中的所有logger：

```properties
logging.level.tomcat=TRACE
```



SpringBoot包含了一些预定义的日志记录组，它们可以直接使用：

| Name | Loggers                                                      |
| :--- | :----------------------------------------------------------- |
| web  | `org.springframework.core.codec`, `org.springframework.http`, `org.springframework.web`, `org.springframework.boot.actuate.endpoint.web`, `org.springframework.boot.web.servlet.ServletContextInitializerBeans` |
| sql  | `org.springframework.jdbc.core`, `org.hibernate.SQL`, `org.jooq.tools.LoggerListener` |

## 6、自定义log配置

springBoot的自动配置会检测类路径的类库来自动激活相应的日志系统，可以提供一个合适的配置来进一步的自定义配置。这个配置应该在 `classpath` 的根目录或者被属性`logging.config` 指定的目录。

你可以使用 `org.springframework.boot.logging.LoggingSystem` 系统属性来强制Spring boot使用一个特指的日志系统。这个值应该是日志系统具体实现的全限定类名。你也可以将这个值设置为none，表示完全关闭日志配置。

> 因为loggin是在 `ApplicationContext` 创建之前初始化的，所以不能在 `@Configuration` 配置的文件中控制日志。自定义日志系统或者关闭它的唯一方式是通过系统属性（system properties）。

依赖于你的日志系统，以下的文件会被加载：

| Logging System          | Customization                                                |
| :---------------------- | :----------------------------------------------------------- |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml`, or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

> 官方推荐我们使用 `-spring` 变体作为日志的配置名（例如：使用`logback-spring.xml`而不是`logback.xml`）->原因：可以使用 `logback` 的扩展功能，详见7。如果你使用标准的配置位置，Spring将不能完全控制日志的初始化。

为了方便自定义log配置，以下是Spring `Environment` 转为`系统属性（System properties）`的一些属性：

| Spring `Environment`                  | 系统属性                          | 注释                                                         |
| :------------------------------------ | :-------------------------------- | :----------------------------------------------------------- |
| `logging.exception-conversion-word`   | `LOG_EXCEPTION_CONVERSION_WORD`   | 记录异常时使用的转换字。                                     |
| `logging.file.clean-history-on-start` | `LOG_FILE_CLEAN_HISTORY_ON_START` | 是否在启动时清除存档日志文件（如果启用了LOG_FILE）。（仅默认登录设置支持。） |
| `logging.file.name`                   | `LOG_FILE`                        | 如果定义，它将在默认日志配置中使用。                         |
| `logging.file.max-size`               | `LOG_FILE_MAX_SIZE`               | 日志文件的最大大小（如果启用了LOG_FILE）。（仅默认登录设置支持。） |
| `logging.file.max-history`            | `LOG_FILE_MAX_HISTORY`            | 要保留的最大归档日志文件数（如果启用了LOG_FILE）。（仅默认登录设置支持。） |
| `logging.file.path`                   | `LOG_PATH`                        | 如果定义，它将在默认日志配置中使用。                         |
| `logging.file.total-size-cap`         | `LOG_FILE_TOTAL_SIZE_CAP`         | 要保留的日志备份的总大小（如果启用了LOG_FILE）。（仅默认登录设置支持。） |
| `logging.pattern.console`             | `CONSOLE_LOG_PATTERN`             | 控制台上要使用的日志模式（stdout）。（仅默认登录设置支持。） |
| `logging.pattern.dateformat`          | `LOG_DATEFORMAT_PATTERN`          | 记录日期格式的附加模式。（仅默认登录设置支持。）             |
| `logging.pattern.file`                | `FILE_LOG_PATTERN`                | 文件中使用的日志模式（如果`LOG_FILE`已启用）。（仅默认登录设置支持。） |
| `logging.pattern.level`               | `LOG_LEVEL_PATTERN`               | 呈现日志级别时使用的格式（默认`%5p`）。（仅默认登录设置支持。） |
| `logging.pattern.rolling-file-name`   | `ROLLING_FILE_NAME_PATTERN`       | 过渡日志文件名的模式（默认`${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz`）。（仅默认登录设置支持。） |
| `PID`                                 | `PID`                             | 当前进程ID（如果可能，并且尚未将其定义为OS环境变量，则发现）。 |

当解析配置文件时，所有支持的日志系统都能够查询系统属性。可以看看spring-boot.jar中的配置样例：

- [Logback](https://github.com/spring-projects/spring-boot/tree/v2.2.4.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/defaults.xml)
- [Log4j 2](https://github.com/spring-projects/spring-boot/tree/v2.2.4.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/log4j2/log4j2.xml)
- [Java Util logging](https://github.com/spring-projects/spring-boot/tree/v2.2.4.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/java/logging-file.properties)

> 如果你想要在一个日志属性中使用占位符，应该使用`Spring Boot` 的语法而不是日志框架的语法。特别地，如果使用`Logback` ，你应该使用 `:` 作为属性名和默认值的分隔符，而不是使用 `-`。

## 7、Logback扩展

SpringBoot包含了许多对 `Logback` 日志框架的扩展，可以帮助我们进行高级的配置。需要注意的是这些扩展功能只能在 `logback-spring.xml` 配置文件中使用。

> <span style="color:green">因为 `logback.xml` 配置文件加载时间过早，所以不能在其中使用扩展功能。只能在 `logback-spring.xml` 中使用或者定义 `logging.config` 属性。</span>

> <span style="color:#B8860B">这些扩展不能和`Logback`的 `configuration scanning` 一起使用。</span>

### 7.1 Profile配置

使用 `<springProfile>` 标签，你可以根据需要不同的环境需要包括或排除某些配置。

使用 `name` 属性指定使用哪个配置文件。`name` 属性可以是一个简单的profile名称，也可以是复杂的表达式。如下例：

```xml
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev | staging">
    <!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
    <!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

### 7.2 Environment属性

`<springProperty>` 标签让你可以将Spring environment 中的属性暴露给 `Logback` 使用。如果你想要在`Logback` 的配置中访问 `application.properties` 文件中的值，这是十分方便的。

该标签的工作方式与 `Logback` 的标准标签 `<property>` 类似。不同的是，你不需要指定一个直接的 `value` ，只需要指定 `source` 属性（这个属性来自于Environment）。

如果你需要将属性存储在 `local` 范围以外的地方，可以使用 `scope` 属性。

如果需要一个默认值(以防该属性没有在 environment 中设置)，可以使用 `defaultValue` 属性。

下面的例子显示了如何在 Logback 中使用这些属性：

```xml
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
        defaultValue="localhost"/>
<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
    <remoteHost>${fluentHost}</remoteHost>
    ...
</appender>
```

> <span style="color:green">`source` 必须以短横线命名的方式指定（例如：`my.property-name`）。虽然属性可能是以松散规则添加到 `Environment`</span>

## 8、SLF4j的使用

### 8.1 如何在系统中使用SLF4j

[官方文档](http://www.slf4j.org/)

在开发的时候，日志记录方法的调用，不应该来直接调用日志的实现类，而是**调用日志抽象层里面的方法**；

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

![](https://gitee.com/yanjundong97/picBed/raw/master/images/SLF4j使用.png)

如上图所示，如果要使用 `logback` 作为日志实现类，需要导入`slf4-api.jar` 和`logback-classic.jar` 、`logback-core.jar` 。如果需要使用 `log4j` 作为日志实现类，需要导入`slf4-api.jar` 、`log4j.jar` 和一个适配器`slf4j-log412.jar` 。其余类似。

> 每一个日志的实现框架都有自己的配置文件。使用slf4j以后，**配置文件还是做成日志实现框架自己本身的配置文件；**

### 8.2 其余问题

框架都有着默认的日志使用框架，比如Hibernate默认使用`jboss-logging`作为日志抽象层，那如果现在在项目中统一都要使用 `SLF4j` 和 `logback` ，该如何呢？

![](https://gitee.com/yanjundong97/picBed/raw/master/images/legacy.png)

如上图所示，如果使用`Commons logging Api` 的框架要改成使用`SLF4j`作为日志抽象层，则需要：

1. 移除`commons-logging` 的jar包
2. 导入中间包 `jcl-cover-slf4j.jar` （该包和`commons-logging` 包的类名、方法全部一样，只是实现不同）
3. 导入`slf4-api.jar` 和`logback-classic.jar` 、`logback-core.jar` 

## 9、Logback.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
scan：当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
scanPeriod：设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒当scan为true时，此属性生效。默认的时间间隔为1分钟。
debug：当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。
-->
<configuration scan="false" scanPeriod="60 seconds" debug="false">
    <!-- 定义日志的根目录 -->
    <property name="LOG_HOME" value="/Users/yanjundong/Desktop/log" />
    <!-- 定义日志文件名称 -->
    <property name="appName" value="atguigu-springboot"></property>
    <!-- ch.qos.logback.core.ConsoleAppender 表示控制台输出 -->
    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <!--
        日志输出格式：
			%d表示日期时间，
			%thread表示线程名，
			%-5level：级别从左显示5个字符宽度
			%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
			%msg：日志消息，
			%n是换行符
        -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </layout>
    </appender>

    <!-- 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件 -->  
    <appender name="appLogAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 指定日志文件的名称 -->
        <file>${LOG_HOME}/${appName}.log</file>
        <!--
        当发生滚动时，决定 RollingFileAppender 的行为，涉及文件移动和重命名
        TimeBasedRollingPolicy： 最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责出发滚动。
        -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--
            滚动时产生的文件的存放位置及文件名称 %d{yyyy-MM-dd}：按天进行日志滚动 
            %i：当文件大小超过maxFileSize时，按照i进行文件滚动
            -->
            <fileNamePattern>${LOG_HOME}/${appName}-%d{yyyy-MM-dd}-%i.log</fileNamePattern>
            <!-- 
            可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每天滚动，
            且maxHistory是365，则只保存最近365天的文件，删除之前的旧文件。注意，删除旧文件是，
            那些为了归档而创建的目录也会被删除。
            -->
            <MaxHistory>365</MaxHistory>
            <!-- 
            当日志文件超过maxFileSize指定的大小是，根据上面提到的%i进行日志文件滚动 注意此处配置SizeBasedTriggeringPolicy是无法实现按文件大小进行滚动的，必须配置timeBasedFileNamingAndTriggeringPolicy
            -->
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <!-- 日志输出格式： -->     
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [ %thread ] - [ %-5level ] [ %logger{50} : %line ] - %msg%n</pattern>
        </layout>
    </appender>

    <!-- 
		logger主要用于存放日志对象，也可以定义日志类型、级别
		name：表示匹配的logger类型前缀，也就是包的前半部分
		level：要记录的日志级别，包括 TRACE < DEBUG < INFO < WARN < ERROR
		additivity：作用在于children-logger是否使用 rootLogger配置的appender进行输出，
		false：表示只用当前logger的appender-ref，true：
		表示当前logger的appender-ref和rootLogger的appender-ref都有效
    -->
    <!-- hibernate logger -->
    <logger name="com.atguigu" level="debug" />
    <!-- Spring framework logger -->
    <logger name="org.springframework" level="debug" additivity="false"></logger>



    <!-- 
    root与logger是父子关系，没有特别定义则默认为root，任何一个类只会和一个logger对应，
    要么是定义的logger，要么是root，判断的关键在于找到这个logger，然后判断这个logger的appender和level。 
    -->
    <root level="info">
        <appender-ref ref="stdout" />
        <appender-ref ref="appLogAppender" />
    </root>
</configuration> 
```

# 五、国际化

国际化，也叫 `i18n` ，这是因为国际化英文是 internationalization ，在 i 和 n 之间有 18 个字母，所以叫 i18n。我们的应用如果做了国际化就可以在不同的语言环境下，方便的进行切换，最常见的就是中文和英文之间的切换。

SpringBoot支持国际化，帮助满足不同语言需求的用户。

在Spring中，就通过 `AcceptHeaderLocaleContextResolver` 对国际化提供了支持。

## 1、基本使用

SpringBoot对于国际化的支持，默认是通过 `AcceptHeaderLocaleResolver` 解析器来完成的，这个解析器是通过请求头中的 `Accept-Language` 字段来判断当前请求所属环境的。

默认的国际化配置是放在resource目录下，下面我们写几个测试文件：

![](https://gitee.com/yanjundong97/picBed/raw/master/images/国际化配置.png)

```properties
#messages.properties
user.name=名字
```

```properties
#messages_zh_CN.properties
user.name=name
```

```properties
##messages_en_US.properties
user.name=姓名~
```

> 需要注意的是：
>
> - 在创建上面三个文件时，IDE编译器会自动把添加到`Resource Bundle messages`，这个不需要我们手动去创建；
>
> - 默认的文件名称是 `messages` ，如果没有配置，就不能更改。
>
>   ![img](https://gitee.com/yanjundong97/picBed/raw/master/images/WX20200306-102146.png)



配置完成后，我们就可以直接开始使用了。在需要使用值的地方，直接注入 MessageSource 实例即可。如下例：

```java
/*HelloController.java*/
@RestController
public class HelloController {
    @Autowired
    private MessageSource messageSource;

    @GetMapping("/hello")
    public String hello() {
        return messageSource.getMessage("user.name", null, LocaleContextHolder.getLocale());
    }
}
```

> `getMessage`  方法：
>
> - 第一个参数是要获取变量的 key
>
> - 第二个参数是如果 value 中有占位符，可以从这里传递参数进去
> - 第三个参数传递一个 Locale 实例即可，这里传入了当前的语言环境。

下面是使用 `Postman` 对这个接口的测试：

![](https://gitee.com/yanjundong97/picBed/raw/master/images/zh-CN.png)



![](https://gitee.com/yanjundong97/picBed/raw/master/images/en-US.png)

配置文件中`messages.properties` 是默认配置，当没有语言匹配时，就会返回其中的内容（我们上面没有配置 `zh_TW` 语言）：

![](https://gitee.com/yanjundong97/picBed/raw/master/images/默认配置.png)

## 2、自定义切换

参数可以当成普通参数放在地址栏上，通过如下配置可以实现我们的需求。

```java
/*WebConfig.java*/
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        LocaleChangeInterceptor interceptor = new LocaleChangeInterceptor();
        interceptor.setParamName("lang");
        registry.addInterceptor(interceptor);
    }

    @Bean
    LocaleResolver localeResolver() {
        SessionLocaleResolver localeResolver = new SessionLocaleResolver();
        localeResolver.setDefaultLocale(Locale.SIMPLIFIED_CHINESE);//默认中文简体
        return localeResolver;
    }
}
```

> 在上面的配置中，我们先配了一个 `Bean` ，这个 bean 会替换掉默认的 AcceptHeaderLocaleResolver（因此该bean 的名字必须为localResolver，否则检测不到），不同于 AcceptHeaderLocaleResolver 通过请求头来判断当前的环境信息，SessionLocaleResolver 将客户端的 Locale 保存到 HttpSession 对象中。（这意味着当前环境信息，前端发送一次即可记住，只要 session 有效，浏览器不必再次告诉服务端当前的环境信息）。
>
> 另外还配置了一个拦截器，这个拦截器会拦截请求中 key 为 lang 的参数（不配置的话默认是 locale），这个参数则指定了当前的环境信息。

![](https://gitee.com/yanjundong97/picBed/raw/master/images/WX20200306-132758.png)

在第二次请求时，就不需要携带lang这个参数。

## 3、其他自定义

默认情况下，配置文件是放在 resources 目录下，如果想自定义，也是可以的，例如定义在 resources/i18n 目录下：

![](https://gitee.com/yanjundong97/picBed/raw/master/images/WX20200306-133259.png)

但是，还需要在 `application.properties` 中进行额外配置，否则系统不能找到(注意这是一个相对路径)：

```properties
spring.messages.basename=i18n/messages
```

另外还有一些编码格式的配置等，一般不需要更改这些默认配置，内容如下：

```
spring.messages.cache-duration=3600
spring.messages.encoding=UTF-8
```

`spring.messages.cache-duration` 表示 messages 文件的缓存失效时间，如果不配置则缓存一直有效。

## 4、语言简称表

| 语言               | 简称  |
| :----------------- | :---- |
| 简体中文(中国)     | zh_CN |
| 繁体中文(中国台湾) | zh_TW |
| 繁体中文(中国香港) | zh_HK |
| 英语(中国香港)     | en_HK |
| 英语(美国)         | en_US |
| 英语(英国)         | en_GB |
| 英语(全球)         | en_WW |
| 英语(加拿大)       | en_CA |
| 英语(澳大利亚)     | en_AU |
| 英语(爱尔兰)       | en_IE |
| 英语(芬兰)         | en_FI |
| 芬兰语(芬兰)       | fi_FI |
| 英语(丹麦)         | en_DK |
| 丹麦语(丹麦)       | da_DK |
| 英语(以色列)       | en_IL |
| 希伯来语(以色列)   | he_IL |
| 英语(南非)         | en_ZA |
| 英语(印度)         | en_IN |
| 英语(挪威)         | en_NO |
| 英语(新加坡)       | en_SG |
| 英语(新西兰)       | en_NZ |
| 英语(印度尼西亚)   | en_ID |
| 英语(菲律宾)       | en_PH |
| 英语(泰国)         | en_TH |
| 英语(马来西亚)     | en_MY |
| 英语(阿拉伯)       | en_XA |
| 韩文(韩国)         | ko_KR |
| 日语(日本)         | ja_JP |
| 荷兰语(荷兰)       | nl_NL |
| 荷兰语(比利时)     | nl_BE |
| 葡萄牙语(葡萄牙)   | pt_PT |
| 葡萄牙语(巴西)     | pt_BR |
| 法语(法国)         | fr_FR |
| 法语(卢森堡)       | fr_LU |
| 法语(瑞士)         | fr_CH |
| 法语(比利时)       | fr_BE |
| 法语(加拿大)       | fr_CA |
| 西班牙语(拉丁美洲) | es_LA |
| 西班牙语(西班牙)   | es_ES |
| 西班牙语(阿根廷)   | es_AR |
| 西班牙语(美国)     | es_US |
| 西班牙语(墨西哥)   | es_MX |
| 西班牙语(哥伦比亚) | es_CO |
| 西班牙语(波多黎各) | es_PR |
| 德语(德国)         | de_DE |
| 德语(奥地利)       | de_AT |
| 德语(瑞士)         | de_CH |
| 俄语(俄罗斯)       | ru_RU |
| 意大利语(意大利)   | it_IT |
| 希腊语(希腊)       | el_GR |
| 挪威语(挪威)       | no_NO |
| 匈牙利语(匈牙利)   | hu_HU |
| 土耳其语(土耳其)   | tr_TR |
| 捷克语(捷克共和国) | cs_CZ |
| 斯洛文尼亚语       | sl_SL |
| 波兰语(波兰)       | pl_PL |
| 瑞典语(瑞典)       | sv_SE |
| 西班牙语(智利)     | es_CL |

# 六、JSON

SpringBoot提供了3种 JSON 映射库的整合。

- Gson
- Jackson
- JSON-B

Jackson 是首选的默认库。

# 七、开发Web应用程序

SpringBoot 工程中，可以使用 Tomcat，Jetty，Undertow或Netty创建独立的HTTP服务器。大多数的web应用程序都使用 `spring-boot-starter-web` 来快速启动。也可以使用  `spring-boot-starter-webflux` 来构建一个反应式的web应用。

## 1、Spring MVC

`Spring MVC` 是一个强大的 mvc 框架，可以通过创建一个特定的 `@Controller`或`@RestController`bean来处理传入的 HTTP 请求，控制器中的方法通过使用 `@RequestMapping` 注解来映射HTTP 请求。

```java
@RestController
@RequestMapping(value="/users")
public class MyRestController {

    @RequestMapping(value="/{user}", method=RequestMethod.GET)
    public User getUser(@PathVariable Long user) {
        // ...
    }

    @RequestMapping(value="/{user}/customers", method=RequestMethod.GET)
    List<Customer> getUserCustomers(@PathVariable Long user) {
        // ...
    }

    @RequestMapping(value="/{user}", method=RequestMethod.DELETE)
    public User deleteUser(@PathVariable Long user) {
        // ...
    }

}
```

### 1.1 Spring MVC 自动配置

Spring Boot 为Spring MVC 提供了自动配置，这在Spring默认配置的基础上添加了很多新的功能：

- 包含`ContentNegotiatingViewResolver`和`BeanNameViewResolver`。
- 支持提供静态资源，包括对WebJars的支持。
- `Converter`，`GenericConverter`和`Formatter` bean类的自动注册。
- 支持`HttpMessageConverters`
- 自动注册`MessageCodesResolver`
- 静态`index.html`支持。
- 定制`Favicon`支持
- 自动使用`ConfigurableWebBindingInitializer`bean

### 1.2 HttpMessageConverters

` Spring MVC` 使用 `HttpMessageConverters` 接口转换 HTTP 请求和响应。这其中提供了很多优秀的默认配置。例如：使用 `Jackson` 包自动地把对象转换为 JSON，或者使用 `Jackson XML extension` 或 `JAXB` 把对象转化为 XML。默认情况下，String使用 `UTF-8`编码方式。

如果需要添加或自定义转换器，可以使用 Spring Boot 的 `HttpMessageConverters` 类，就像下面一样：

```java
import org.springframework.boot.autoconfigure.http.HttpMessageConverters;
import org.springframework.context.annotation.*;
import org.springframework.http.converter.*;

@Configuration(proxyBeanMethods = false)
public class MyConfiguration {

    @Bean
    public HttpMessageConverters customConverters() {
        HttpMessageConverter<?> additional = ...
        HttpMessageConverter<?> another = ...
        return new HttpMessageConverters(additional, another);
    }

}
```

在上下文中存在的任何 `HttpMessageConverters` bean都会被添加到 转换器列表。当然也可以使用相同的方法覆盖默认的转换器。

### 1.3 自定义JSON的序列化与反序列化

如果你使用 Jackson 序列化和反序列化 JSON 数据，你可能需要写你自己的  `JsonSerializer` and `JsonDeserializer` 类。但是在 Spring Boot 中，你能够直接在 `JsonSerializer`, `JsonDeserializer` or `KeyDeserializer` 的实现上使用  `@JsonComponent` 注解。你也能够在包含  serializers/deserializers的 类上使用它，类似于：

```java
import java.io.*;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.*;
import org.springframework.boot.jackson.*;

@JsonComponent
public class Example {

    public static class Serializer extends JsonSerializer<SomeObject> {
        // ...
    }

    public static class Deserializer extends JsonDeserializer<SomeObject> {
        // ...
    }

}
```

### 1.4 静态资源

默认情况下，Spring Boot从`classpath`中名为 `/static`（`/public`或`/resources`或`/META-INF/resources`）或者从  `ServletContext` 的根路径中寻找静态资源。因此你可以添加自己的 `WebMvcConfigurer` 类并覆写 `addResourceHandlers` 方法来修改这种行为。

默认情况下，资源映射到`/**`，但是您可以使用`spring.mvc.static-path-pattern`属性进行调整。例如，将所有资源重新定位到`/resources/**`以下位置即可：

```properties
spring.mvc.static-path-pattern=/resources/**
```

也可以使用`spring.resources.static-locations`属性来自定义静态资源位置。

### 1.5 路径匹配

Spring MVC 能够通过查看请求路径并将其匹配到应用程序中定义的映射（例如，在controller方法中的`@GetMapping` 注解）来将传入的 HTTP 请求映射到处理程序。

默认情况下，Spring Boot 禁用后缀匹配模式，这意味着像 `"GET /projects/spring-boot.json"` 这样的请求将不会与  `@GetMapping("/projects/spring-boot")` 映射匹配。这被认为是 Spring MVC 程序的最佳实践。过去，这个功能主要被使用在不能正确发送 `Accept` 请求头的 HTTP 客户端。需要我们需要确保发送正确的 `Content Type` 给客户端。但是现在，内容协商已变得十分可靠。

除了上面后缀匹配的方法，还可以通过查询参数的方法确保像  `"GET /projects/spring-boot?format=json"` 的请求映射到 `@GetMapping("/projects/spring-boot")` 。

当然，也可以通过配置使得应用程序使用后缀匹配。

```properties
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-suffix-pattern=true
```

### 1.6 ConfigurableWebBindingInitializer

### 1.7 模板引擎

Spring MVC 支持大量的模板引擎，包括 `Thymeleaf`, `FreeMarker`, and `JSP` 。

Spring Boot 对以下模板引擎提供了自动配置的支持：

- `FreeMarker`
- `Groovy`
- `Thymeleaf`
- `Mustache`

> 如果可能，应该避免使用 JSP。

在使用模板引擎时，会自动从  `src/main/resources/templates` 处提取模板。

### 1.8 错误处理

默认情况下，Spring Boot 提供了一个 `/error` 映射来处理所有的错误，并且已经在 servlet 容器中注册为全局的错误页面。对于 `machine client` ，它会以JSON形式返回带有 ==错误细节、HTTP状态码和异常信息== 的响应。对于 `browser client` ，有一个被以上数据渲染的HTML格式的错误试图（可以通过添加解析为 `/error` 的 `view` 自定义）。

为了完全替换默认行为，你可以实现 `ErrorController` 接口，并注册该类型的bean，或者添加`ErrorAttributes` 类型的 bean 以使用现有机制但是替换其内容。

> 在自定义实现 `ErrorController` 类时， `BasicErrorController` 能够被使用作为一个基础类。

你也能够创建一个添加了 `@ControllerAdvice` 注解的类来自定义JSON 文档，以针对特定的 `controller`、`/` 或`异常类型`。类如：

```java
@ControllerAdvice(basePackageClasses = AcmeController.class)
public class AcmeControllerAdvice extends ResponseEntityExceptionHandler {

    @ExceptionHandler(YourException.class)
    @ResponseBody
    ResponseEntity<?> handleControllerException(HttpServletRequest request, Throwable ex) {
        HttpStatus status = getStatus(request);
        return new ResponseEntity<>(new CustomErrorType(status.value(), ex.getMessage()), status);
    }

    private HttpStatus getStatus(HttpServletRequest request) {
        Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
        if (statusCode == null) {
            return HttpStatus.INTERNAL_SERVER_ERROR;
        }
        return HttpStatus.valueOf(statusCode);
    }

}
```

### 1.9 CORS支持

`Cross-origin resource sharing` （CORS）是由大多数浏览器实现的w3c标准，可让你灵活的指定允许哪种类型的跨域请求。

从4.2版本开始，Spring MVC支持CORS。在Spring Boot应用程序中，使用添加了 [`@CrossOrigin`](https://docs.spring.io/spring/docs/5.2.3.RELEASE/javadoc-api/org/springframework/web/bind/annotation/CrossOrigin.html) 注解的controller方法不需要任何特定的配置就可以实现 CORS 配置。

```java
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin
    @GetMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

能够通过注册一个带有 `addCorsMappings(CorsRegistry)` 方法的 `WebMvcConfigurer` bean定义全局的CORS配置。类如：

```java
@Configuration(proxyBeanMethods = false)
public class MyConfiguration {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**");
            }
        };
    }
}
```

## 2、内置的Servlet容器支持

Spring Boot包含了对内置 Tomcat、Jetty和Undertow的支持。

# 八、RSocket

# 九、安全

## 1、Spring Security

- [`Spring Security `原理分析](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247485379&idx=1&sn=023d15f7d1944a794db3a302e1a87a12&scene=21#wechat_redirect)
- [`Spring Security` 整合 `JWT`](https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247487270&idx=2&sn=dcc3bb1660145cb2ff3ce3cecd85b012&chksm=e9c35d46deb4d4500f2b918ec0d5168f4e6fe929d71be094ff5568c43d1135aad570021adb2d&scene=21#wechat_redirect)

- 角色继承：

  ```java
  @Bean
  RoleHierarchy roleHierarchy() {
      RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();
      String hierarchy = "ROLE_dba > ROLE_admin \n ROLE_admin > ROLE_user";
      roleHierarchy.setHierarchy(hierarchy);
      return roleHierarchy;
  }
  ```


## 2、OAuth2

# 十、使用SQL数据库

在Spring框架中提供了对SQL数据库的额外支持，包括使用 `JdbcTemplate`  的JDBC 连接和完全的`ORM(object relational mapping)`技术，例如`Hibernate`。 Spring Data提供了更高级的功能：直接从接口创建 `Repository` 实现，并且从你方法的名称中生成查询。

## 1、连接到生产数据库

Spring Boot 使用以下的算法来选择特定的实现：

1. 如果 `HikariCP` 可用，总是选择它。
2. 否则，如果 `Tomcat pooling DataSource` 可用，就使用它。
3. 如果上述两者都不可用，但是 `Commons DBCP2` 是可用的，就是用它。

如果你使用 `spring-boot-starter-jdbc` 或者 `spring-boot-starter-data-jpa`  starter，就自动获得了  `HikariCP` 依赖。

> 可以通过配置 `spring.datasource.type` 属性来指定使用的连接池，这样就绕过了上述算法。如果应用程序是运行在 Tomcat 容器中，这是尤其重要的。因为 `tomcat-jdbc` 被提供。

`DataSource` 配置能够被额外的配置文件控制，例如，你可以像下面这样配置：

```properties
spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

> 至少应该指定 `spring.datasource.url` ，否则，Spring Boot 将尝试自动配置一个内置的数据库。
>
> 通常不需要指定 `spring.datasource.driver-class-name` ，因为 Spring Boot 可以从`spring.datasource.url` 中推断中大多数的数据库。

## 2、使用 `JdbcTemplate`

在Spring中，`JdbcTemplate`和`NamedParameterJdbcTemplate`类是自动配置的，可以用 `@Autowire` 来把它们注入自己的bean。

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final JdbcTemplate jdbcTemplate;

    @Autowired
    public MyBean(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    // ...

}
```

可以使用 `spring.jdbc.template.*` 来自定义 Template 的某些属性，例如：

```properties
spring.jdbc.template.max-rows=500
```

## 3、JPA和Spring Data JPA

`Java Persistence API` 是一个标准的技术，让你能够映射对象到关系型数据库。`spring-boot-starter-data-jpa` 提供了一个快速启动的方法。它提供了以下关键的依赖：

- Hibernate：最流行的 JPA 实现之一。
- Spring Data JPA：使基于 JAP的repository实现更容易。
- Spring ORMs：Spring Framework提供的核心ORM支持。

# 十一、使用NoSQL数据库

Spring Data提供了额外的项目来帮助你连接各种各样的 NoSQL数据库，包括：

- MongoDB
- Neo4J
- Elasticsearch
- Solr
- Redis
- Geode
- Cassandra
- Couchbase
- LDAP

# 十二、Caching

# 十三、Messaging

# 十四、RestTemplate

# 十七、发送电子邮件

`Spring` 框架提供了一种使用 `JavaMailSender` 接口的简单抽象方法发送电子邮件，而 `SpringBoot` 为其提供了自动配置。

首先我们需要导入 `spring-boot-starter-mail` 包。

```xml
<!--mail-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

特别注意，因为某些默认超时值是无限的，所以需要重新配置，以避免线程被一个无相应的邮箱服务器阻塞，如下面的例子所示：

```properties
spring.mail.properties.mail.smtp.connectiontimeout=5000
spring.mail.properties.mail.smtp.timeout=3000
spring.mail.properties.mail.smtp.writetimeout=5000
```

我们要知道，如果要从QQ邮箱发送一封邮件给163邮箱，并不是两个用户之间直接发送的，而是如下所示，用户A把邮件发送给QQ邮箱服务器，然后QQ邮箱服务器发送给163邮箱服务器，最后用户B上线之后再获取这封邮件。

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ20200328-214337.png)

所以，我们还需要配置QQ邮箱的主机，你的邮箱账户的账号、密码：

```yml
spring:
  #mail的配置
  mail:
    properties:
      smtp:
        connectiontimeout: 5000
        timeout: 3000
        writetimeout: 5000
    username: #你的邮箱账号
    password: #授权码
    host: smtp.qq.com
```

> 注意：上面的 `password` 并不是登录时的密码，获取方式如下：
>
> ![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ20200328-215036.png)



## 1、`SimpleMailMessage`用法

接下来我们就可以发送电子邮件

```java
@SpringBootTest
class UmserverApplicationTests {
	@Autowired
	JavaMailSender mailSender;
	
	@Test
	public void testEmail() {
		SimpleMailMessage msg = new SimpleMailMessage();
		msg.setTo("yanjundong97@163.com");
		msg.setSubject("一封邮件");
		msg.setText("一封来自QQ邮箱的信件");
		msg.setFrom("yanjundong97@qq.com");
		mailSender.send(msg);

	}
}
```

## 2、 `MimeMessageHelper` 用法

有时候我们需要发送复杂的邮件，例如在邮件中加入附件，或者加入 `Html` 标签，那么就可以使用 `MimeMessageHelper` 。

### 2.1 带有附件和Html

```java
@SpringBootTest
class UmserverApplicationTests {

	@Autowired
	JavaMailSender mailSender;
	
	@Test
	public void testEmail() throws MessagingException {
		MimeMessage message = mailSender.createMimeMessage();
		MimeMessageHelper helper = new MimeMessageHelper(message, true);
		helper.setTo("yanjundong97@163.com");
		helper.setFrom("yanjundong97@qq.com");
		helper.setSubject("MimeMessageHelper的使用");
		helper.setText("<span style='color:red'>MimeMessageHelper的详细使用如下：</span>" +
				"<img src='cid:IMG_2139'>", true);

		//在邮件内容中内置资源
		FileSystemResource res = new FileSystemResource(new File("本机上的图片地址"));
		helper.addInline("IMG_2139", res);

		//给邮件添加附件
		FileSystemResource file = new FileSystemResource(new File("本机上的附件地址"));
		helper.addAttachment("CoolImage.jpg", file);

		mailSender.send(message);
	}
}
```

效果如下：

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ20200328-222114.png)

### 2.2 使用模板库创建邮件内容

在企业应用中，开发人员通常不会采用上面的那些方法来生成电子邮件内容，原因如下：

- 用Java代码创建基于HTML的电子邮件内容很繁琐且容易出错；
- 显示逻辑和业务逻辑之间没有明确的区分；
- 更改电子邮件内容的显示结构时需要编写Java代码，重新编译，重新部署等。

解决这个问题的通常办法是使用一个模版库（例如` FreeMarker`）来定义邮件内容的显示结构。这时代码的任务就只剩下产生数据，加入到邮件模版中，然后发送邮件。

# 二十一、任务

## 1、异步任务

**同步任务**就是，A调用了B方法，必须要得B执行完了之后才能执行，如果B方法执行的很慢，A页必须等待。

鉴于这种方法有时候会很低效，所以就有了**异步执行**，A调用B方法之后，会再开一个线程去执行B，不会影响A的执行。



在SpringBoot中要想实现异步任务十分简单，接下来我们先写一个不是异步方法的代码，

```java
@Service
public class AsyncService {
  
    public void hello() throws InterruptedException {
        Thread.sleep(5000);	//测试方法，我们让线程失眠5秒
        System.out.println("处理数据中...");
    }

}
```

编写`controller`层如下：

```java
@RestController
public class AsyncController {

    @Autowired
    AsyncService asyncService;

    @GetMapping("/hello")
    public String hello() throws InterruptedException {
        asyncService.hello();	//调用同步方法
        return "hello";
    }

}
```

运行项目，在浏览器中输入 `http://localhost:8080/hello` ，可以看到在过了 5 秒之后才会输出 `hello` 。

 现在我们来更改 `AsyncService.java` 中的代码：

```java
@Service
public class AsyncService {
    //告诉Spring这是一个异步方法
    @Async
    public void hello() throws InterruptedException {
        Thread.sleep(5000);
        System.out.println("处理数据中...");
    }
}
```

并且在 `UmserverApplication.java` 中开启异步注解:

```java
@SpringBootApplication
@EnableAsync
public class UmserverApplication {

	public static void main(String[] args) {
		SpringApplication.run(UmserverApplication.class, args);
	}

}
```

再次执行，就可以马上看到 `hello` ，但是在 5 秒之后控制台输出 `处理数据中...` 。

### 1.1 配置

> 如果你在 `context` 中定义了一个 `Executor` ，那么异步任务就会使用它。

线程池使用8个核心线程，当然也会根据任务的多少增加或者减少。这些默认的配置能够使用 `spring.task.execution` 命名空间来调整，例如：

```properties
spring.task.execution.pool.max-size=16
spring.task.execution.pool.queue-capacity=100
spring.task.execution.pool.keep-alive=10s
```

上面的配置使线程池使用了一个受限的 `queue` ，当 `queue` 满的时候（达到了100个任务），线程数就会增加，但是最大也只会到 16 个。线程空闲10 秒（而不是默认的 60 秒）时，线程就会被回收。

## 2、定时任务

要使用定时任务也是十分的简单，如下：

在方法上添加上 `@Scheduled` 注解，

```java
@Service
public class MailSend {
  	/*cron的详细配置请看下面*/
    @Scheduled(cron = "0-4 * * * * 1-6")
    public void hello() throws InterruptedException {
        System.out.println("hello...");
    }
}
```

并且在 `UmserverApplication.java` 中添加上 `@EnableScheduling` 注解即可。

```java
@SpringBootApplication
@EnableAsync
@EnableScheduling
public class UmserverApplication {

	public static void main(String[] args) {
		SpringApplication.run(UmserverApplication.class, args);
	}

}
```

启动项目后，在每分钟的 `0-4` 秒时控制台会打印 `hello...`。



关于 `cron` 的值：

| 字段 | 允许值 | 允许的特殊字符  |
| ---- | ------ | --------------- |
| 秒   | 0-59   | , - * /         |
| 分   | 0-59   | , - * /         |
| 小时 | 0-23   | , - * /         |
| 日期 | 1-31   | , - * ? / L W C |
| 月份 | 1-12   | , - * /         |
| 星期 | 1-7    | , - * ? / L C # |

| 特殊字符 | 代表含义                   |
| -------- | -------------------------- |
| ,        | 枚举                       |
| -        | 区间                       |
| *        | 任意                       |
| /        | 步长                       |
| ?        | 日/星期冲突匹配            |
| L        | 最后                       |
| W        | 工作日                     |
| C        | 和calendar联系后计算过的值 |
| #        | 星期， 4#2 ，第2个星期四   |

## 3、关于异步任务中的ThreadPoolTaskExecutor

`ThreadPoolTaskExecutor` 和 `ThreadPoolExecutor` ？

首先 `ThreadPoolTaskExecutor`  是 Spring core包中的，而 `ThreadPoolExecutor` 是 JDK 中的。

前者是对后者的封装处理。

### 3.1 ThreadPoolExecutor

我们先来了解下 `ThreadPoolExecutor`，首先在[Java——多线程](https://blog.csdn.net/weixin_40971059/article/details/104831203) 中说了，常用的线程实现类包括：

- `FixedThreadPool`：线程数固定的线程池；
- `CachedThreadPool`：线程数根据任务动态调整的线程池；
- `SingleThreadExecutor`：仅单线程执行的线程池。

使用这些实现类的方法也会类似于：

```java
ExecutorService executor = Executors.newFixedThreadPool(3);
```

但如果我们打开 `Executors` 类的源码，搜索一下 `ThreadPoolExecutor` ，就可以发现，上面三个实现类都是使用 `ThreadPoolExecutor` 实现的。

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ20200329-124030.png)

![](SpringBoot-img/QQ20200329-123916.png)



最后来看下该类的结构，可以知道祖类为 `Executor` 接口。

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ20200329-130215.png)

### 3.2 ThreadPoolTaskExecutor

来看一下 `ThreadPoolTaskExecutor` 结构，组类也是调用 `Executor` 接口：

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ20200329-130215.png)

再来看一下源码：

```java
public class ThreadPoolTaskExecutor extends ExecutorConfigurationSupport implements AsyncListenableTaskExecutor, SchedulingTaskExecutor {
    private final Object poolSizeMonitor = new Object();
    private int corePoolSize = 1;
    private int maxPoolSize = 2147483647;
    private int keepAliveSeconds = 60;
    private int queueCapacity = 2147483647;
    private boolean allowCoreThreadTimeOut = false;
    @Nullable
    private TaskDecorator taskDecorator;
    @Nullable
    private ThreadPoolExecutor threadPoolExecutor;	//这里用到了 ThreadPoolExecutor
```

`ThreadPoolTashExecutor`类会根据配置设置  `threadPoolExecutor` 的一些参数，例如：

```java
    //设置线程池维护线程的最小数量
		public void setCorePoolSize(int corePoolSize) {
        synchronized(this.poolSizeMonitor) {
            this.corePoolSize = corePoolSize;
            if (this.threadPoolExecutor != null) {
                this.threadPoolExecutor.setCorePoolSize(corePoolSize);
            }

        }
    }
```

`ThreadPoolExecutor` 池子的处理流程如下：　　

1）当池子大小小于corePoolSize就新建线程，并处理请求。

2）当池子大小等于corePoolSize，把请求放入workQueue中，池子里的空闲线程就去从workQueue中取任务并处理

3）当workQueue放不下新入的任务时，新建线程入池，并处理请求，如果池子大小撑到了maximumPoolSize就用RejectedExecutionHandler来做拒绝处理

4）另外，当池子的线程数大于corePoolSize的时候，多余的线程会等待keepAliveTime长的时间，如果无请求可处理就自行销毁。



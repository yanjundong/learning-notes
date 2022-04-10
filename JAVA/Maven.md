

# Maven

## POM的四个层次

### 超级POM

Maven 在构建过程中有很多默认的设定。例如：源文件存放的目录、测试源文件存放的目录、构建输出的目录……等等。但其实这些要素就是在Maven的**超级 POM**中定义。

关于超级POM，Maven 官网是这样介绍的：

> The Super POM is Maven's default POM. All POMs extend the Super POM unless explicitly set, meaning the configuration specified in the Super POM is inherited by the POMs you created for your projects.
>
> 译文：Super POM 是 Maven 的默认 POM。除非明确设置，否则所有 POM 都扩展 Super POM，这意味着 Super POM 中指定的配置由您为项目创建的 POM 继承。

所以我们自己的 POM 即使没有明确指定一个父工程（父 POM），其实也默认继承了超级 POM。就好比一个 Java 类默认继承了 Object 类。

超级POM的内容如下：

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
 
  <repositories>
    <repository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>
 
  <pluginRepositories>
    <pluginRepository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
      <releases>
        <updatePolicy>never</updatePolicy>
      </releases>
    </pluginRepository>
  </pluginRepositories>
 
  <build>
    <directory>${project.basedir}/target</directory>
    <outputDirectory>${project.build.directory}/classes</outputDirectory>
    <finalName>${project.artifactId}-${project.version}</finalName>
    <testOutputDirectory>${project.build.directory}/test-classes</testOutputDirectory>
    <sourceDirectory>${project.basedir}/src/main/java</sourceDirectory>
    <scriptSourceDirectory>${project.basedir}/src/main/scripts</scriptSourceDirectory>
    <testSourceDirectory>${project.basedir}/src/test/java</testSourceDirectory>
    <resources>
      <resource>
        <directory>${project.basedir}/src/main/resources</directory>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>${project.basedir}/src/test/resources</directory>
      </testResource>
    </testResources>
    <pluginManagement>
      <!-- NOTE: These plugins will be removed from future versions of the super POM -->
      <!-- They are kept for the moment as they are very unlikely to conflict with lifecycle mappings (MNG-4453) -->
      <plugins>
        <plugin>
          <artifactId>maven-antrun-plugin</artifactId>
          <version>1.3</version>
        </plugin>
        <plugin>
          <artifactId>maven-assembly-plugin</artifactId>
          <version>2.2-beta-5</version>
        </plugin>
        <plugin>
          <artifactId>maven-dependency-plugin</artifactId>
          <version>2.8</version>
        </plugin>
        <plugin>
          <artifactId>maven-release-plugin</artifactId>
          <version>2.5.3</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
 
  <reporting>
    <outputDirectory>${project.build.directory}/site</outputDirectory>
  </reporting>
 
  <profiles>
    <!-- NOTE: The release profile will be removed from future versions of the super POM -->
    <profile>
      <id>release-profile</id>
 
      <activation>
        <property>
          <name>performRelease</name>
          <value>true</value>
        </property>
      </activation>
 
      <build>
        <plugins>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-source-plugin</artifactId>
            <executions>
              <execution>
                <id>attach-sources</id>
                <goals>
                  <goal>jar-no-fork</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-javadoc-plugin</artifactId>
            <executions>
              <execution>
                <id>attach-javadocs</id>
                <goals>
                  <goal>jar</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-deploy-plugin</artifactId>
            <configuration>
              <updateReleaseInfo>true</updateReleaseInfo>
            </configuration>
          </plugin>
        </plugins>
      </build>
    </profile>
  </profiles>
 
</project>
```

### 父POM

和 Java 类一样，Maven 中的POM也是 **单继承** 的，如果我们给一个 POM 指定了父 POM，那么继承关系如下：

![image-20220406181521582](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220406181521582.png)

### 当前POM

我们关注和最多使用的一层 POM 文件。

### 有效POM

在 Maven 的 POM 继承关系中，子 POM 可以覆盖父 POM 中的配置；如果子 POM 没有覆盖，那么父 POM 中的配置将会被继承。按照这个规则，继承关系中的所有 POM 叠加到一起，就得到了一个最终生效的 POM，这就是有效 POM（effective POM）。

可以通过命令 `mvn help:effective-pom` 查看有效 POM。



## 属性的声明与引用

### help 插件的各个目标

| 目标                    | 说明                                              |
| ----------------------- | ------------------------------------------------- |
| help:active-profiles    | 列出当前已激活的 profile                          |
| help:all-profiles       | 列出当前工程所有可用 profile                      |
| help:describe           | 描述一个插件和/或 Mojo 的属性                     |
| help:effective-pom      | 以 XML 格式展示有效 POM                           |
| help:effective-settings | 为当前工程以 XML 格式展示计算得到的 settings 配置 |
| **help:evaluate**       | 计算用户在交互模式下给出的 Maven 表达式           |
| help:system             | 显示平台详细信息列表，如系统属性和环境变量        |

### 使用 help:evaluate 查看属性值

定义属性

```xml
<fastjson.version>1.2.62</fastjson.version>
```

使用 `mvn help:evaluate` 查看属性值

```sh
[INFO] Enter the Maven expression i.e. ${project.groupId} or 0 to exit?:
${fastjson.version}
[INFO]                                                                  
1.2.62
```

### 通过Maven访问系统变量

如果是要通过 Java 代码访问系统变量，可以通过如下方法访问：

```java
Properties properties = System.getProperties();
Set<Object> propNameSet = properties.keySet();
for (Object propName : propNameSet) {
    String propValue = properties.getProperty((String) propName);
    System.out.println(propName + " = " + propValue);
}
```



运行结果：

> java.runtime.name = Java(TM) SE Runtime Environment
> sun.boot.library.path = C:\software\Programs\Java\jdk1.8.0_231\jre\bin
> java.vm.version = 25.231-b11
> java.vm.vendor = Oracle Corporation
> java.vendor.url = http://java.oracle.com/
> path.separator = ;
> java.vm.name = Java HotSpot(TM) 64-Bit Server VM
> file.encoding.pkg = sun.io
> user.country = CN
> user.script = 
> sun.java.launcher = SUN_STANDARD
> sun.os.patch.level = 
> java.vm.specification.name = Java Virtual Machine Specification
> ......



**通过Maven 访问：** 

```shell
PS C:\Users\sweet\Documents\JetBrains\IdeaProjects\learnJava\ch8> mvn help:evaluate
[INFO] ---------------------------< com.coder:ch8 >----------------------------
[INFO] No artifact parameter specified, using 'com.coder:ch8:jar:1.0-SNAPSHOT' as project.
[INFO] Enter the Maven expression i.e. ${project.groupId} or 0 to exit?:
${java.vm.version}
[INFO]
25.231-b11
```

### 通过Maven访问系统环境变量

```shell
[INFO] Enter the Maven expression i.e. ${project.groupId} or 0 to exit?:
${env.JAVA_HOME} 
[INFO] 
C:\software\Programs\Java\jdk1.8.0_231
```



### 访问 Project 属性

我们可以使用表达式 `${project.xxx}` 访问当前 POM 中的元素值。

**访问一级标签：**

![image-20220406185515355](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220406185515355.png)

**访问子标签：**

![image-20220406185713297](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220406185713297.png)

**访问列表标签：**

![image-20220406185910916](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220406185910916.png)

### 访问 settings 全局配置

![image-20220407155338654](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220407155338654.png)

### 用途

1. 在**当前 pom.xml** 文件中引用属性
2. 资源过滤功能：在非 Maven 配置文件中引用属性，由 Maven 在处理资源时将引用属性的表达式替换为属性值

## build 标签详解

### 完整的build标签

通过有效 POM 我们能够看到，build 标签的完整配置。

```xml
<build>
    <!----------------------------1、定义约定的目录结构------------------------------------>
    <!--主体源程序的存放目录-->
    <sourceDirectory>
        C:\Users\sweet\Documents\JetBrains\IdeaProjects\learnJava\kaoshi\src\main\java
    </sourceDirectory>
    <!--脚本源程序的存放目录-->
    <scriptSourceDirectory>
        C:\Users\sweet\Documents\JetBrains\IdeaProjects\learnJava\kaoshi\src\main\scripts
    </scriptSourceDirectory>
    <!--测试源程序的存放目录-->
    <testSourceDirectory>
        C:\Users\sweet\Documents\JetBrains\IdeaProjects\learnJava\kaoshi\src\test\java
    </testSourceDirectory>
    <!--主体源程序编译结果输出目录-->
    <outputDirectory>
        C:\Users\sweet\Documents\JetBrains\IdeaProjects\learnJava\kaoshi\target\classes
    </outputDirectory>
    <!--测试源程序编译结果输出目录-->
    <testOutputDirectory>
        C:\Users\sweet\Documents\JetBrains\IdeaProjects\learnJava\kaoshi\target\test-classes
    </testOutputDirectory>
    <!--主体资源文件存放目录-->
    <resources>
        <resource>
            <directory>
                C:\Users\sweet\Documents\JetBrains\IdeaProjects\learnJava\kaoshi\src\main\resources
            </directory>
        </resource>
    </resources>
    <!--测试资源文件存放目录-->
    <testResources>
        <testResource>
            <directory>
                C:\Users\sweet\Documents\JetBrains\IdeaProjects\learnJava\kaoshi\src\test\resources
            </directory>
        </testResource>
    </testResources>
    <!--构建结果输出目录-->
    <directory>
        C:\Users\sweet\Documents\JetBrains\IdeaProjects\learnJava\kaoshi\target
    </directory>
    <!--构建结果的名称-->
    <finalName>kaoshi-1.0-SNAPSHOT</finalName>
    <!----------------------------2、备用插件的管理------------------------------------>
    <!--统一管理插件的版本，类似dependencyManagement标签-->
    <pluginManagement>
        <plugins>
            <plugin>
                <artifactId>maven-antrun-plugin</artifactId>
                <version>1.3</version>
            </plugin>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.2-beta-5</version>
            </plugin>
            <plugin>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>2.8</version>
            </plugin>
            <plugin>
                <artifactId>maven-release-plugin</artifactId>
                <version>2.5.3</version>
            </plugin>
        </plugins>
    </pluginManagement>
    <!----------------------------3、生命周期插件------------------------------------>
    <plugins>
        <plugin>
            <artifactId>maven-clean-plugin</artifactId>
            <version>2.5</version>
            <executions>
                <execution>
                    <id>default-clean</id>
                    <phase>clean</phase>
                    <goals>
                        <goal>clean</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        .
    </plugins>
</build>

```



### 典型应用：指定JDK版本

要想指定Maven在编译时的JDK版本，我们可以通过配置 `Maven的settings.xml` 文件 或  `项目的pom.xml`。

**方法一：**

```xml
<profile>
  <id>jdk-1.8</id>
    
  <activation>
	<activeByDefault>true</activeByDefault>
	<jdk>1.8</jdk>
  </activation>
    
  <properties>
	<maven.compiler.source>1.8</maven.compiler.source>
	<maven.compiler.target>1.8</maven.compiler.target>
	<maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
  </properties>
</profile>
```

**方法二：** 通过配置pom.xml ，直接告诉负责编译操作的 maven-compiler-plugin 插件，需要的编译环境。

```xml
<build>
    <!-- plugins 标签：在构建的时候要用到这些插件！ -->
    <plugins>
        <!-- plugin 标签：这是我要指定的一个具体的插件 -->
        <plugin>
            <!-- 插件的坐标。此处引用的 maven-compiler-plugin 插件不是第三方的，是一个 Maven 自带的插件。 -->
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.1</version>
            
            <!-- configuration 标签：配置 maven-compiler-plugin 插件 -->
            <configuration>
                <!-- 具体配置信息会因为插件不同、需求不同而有所差异 -->
                <source>1.8</source>
                <target>1.8</target>
                <encoding>UTF-8</encoding>
            </configuration>
        </plugin>
    </plugins>
</build>
```



### 典型应用：SpringBoot 定制化打包

 使用 `spring-boot-maven-plugin` 可以将『微服务』导出为一个 jar 包，这个 jar 包可以使用 java -jar xxx.jar 这样的命令直接启动运行。

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
			<version>2.5.5</version>
		</plugin>
	</plugins>
</build>
```



### 典型应用：Mybatis 逆向工程

使用 Mybatis 的逆向工程需要使用如下配置，MBG 插件的特点是需要提供插件所需的依赖：

```xml
<!-- 控制 Maven 在构建过程中相关配置 -->
<build>		
	<!-- 构建过程中用到的插件 -->
	<plugins>
		
		<!-- 具体插件，逆向工程的操作是以构建过程中插件形式出现的 -->
		<plugin>
			<groupId>org.mybatis.generator</groupId>
			<artifactId>mybatis-generator-maven-plugin</artifactId>
			<version>1.3.0</version>
	
			<!-- 插件的依赖 -->
			<dependencies>
				
				<!-- 逆向工程的核心依赖 -->
				<dependency>
					<groupId>org.mybatis.generator</groupId>
					<artifactId>mybatis-generator-core</artifactId>
					<version>1.3.2</version>
				</dependency>
					
				<!-- 数据库连接池 -->
				<dependency>
					<groupId>com.mchange</groupId>
					<artifactId>c3p0</artifactId>
					<version>0.9.2</version>
				</dependency>
					
				<!-- MySQL驱动 -->
				<dependency>
					<groupId>mysql</groupId>
					<artifactId>mysql-connector-java</artifactId>
					<version>5.1.8</version>
				</dependency>
			</dependencies>
		</plugin>
	</plugins>
</build>
```

## 依赖配置补充

### 依赖范围

Maven中共有6个依赖范围：

- compile，这是默认的依赖范围
- provided
- runtime
- test
- system
- import

**import：**

和 Java 类一样，**Maven 也是单继承的**。如果不同体系的依赖信息封装在不同 POM 中了，没办法继承多个父工程，这时就可以使用 import 依赖范围。

典型案例当然是在项目中引入 SpringBoot、SpringCloud 依赖：

```xml
<dependencyManagement>
    <dependencies>
        <!-- SpringCloud 依赖导入 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR9</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>

        <!-- SpringCloud Alibaba 依赖导入 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.2.6.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>

        <!-- SpringBoot 依赖导入 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.3.6.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```



> import 依赖范围使用要求：
>
> - 打包类型必须是 pom
> - 必须放在 dependencyManagement 中



**system:**

假设现在 D:\rep\aaa-1.0-SNAPSHOT.jar 想要引入到我们的项目中，此时我们就可以将依赖配置为 system 范围：

```xml
<dependency>
    <groupId>com.atguigu.maven</groupId>
    <artifactId>atguigu-maven-test-aaa</artifactId>
    <version>1.0-SNAPSHOT</version>
    <systemPath>D:\rep\aaa-1.0-SNAPSHOT.jar</systemPath>
    <scope>system</scope>
</dependency>
```

> 但是很明显：这样引入依赖完全不具有可移植性，所以**不要使用**。
>

**runtime:**

专门用于编译时不需要，但是运行时需要的 jar 包。典型案例是：

```xml
<!--热部署 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

### 可选依赖

```xml
<!--热部署 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

其核心含义是：Project X 依赖 Project A，但是A 中的一部分 X 用不到的代码依赖了 B，那么对 X 来说 B 就是『可有可无』的。

### 版本仲裁

1. 最短路径优先

   ![image-20220408113542648](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220408113542648.png)

2. 路径相同时，先声明者优先。此时 Maven 采纳哪个版本，取决于在 pro29-module-x 中，对 pro30-module-y 和 pro31-module-z 两个模块的依赖哪一个先声明。

   ![image-20220408113554305](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220408113554305.png)

> 在项目正常运行的情况下，jar 包版本可以由 Maven 仲裁，不必我们操心；而发生冲突时如果 Maven 仲裁决定的版本无法满足要求，此时就应该由程序员明确指定 jar 包版本。

## Maven自定义插件

[](./Maven自定义插件.md)

### 参考

- http://heavy_code_industry.gitee.io/code_heavy_industry/pro002-maven/chapter09/verse06.html
- https://www.bilibili.com/video/BV12q4y147e4?p=155&spm_id_from=pageDriver

## 搭建Maven私服：Nexus

[](./搭建Maven私服.md)

### 参考

- http://heavy_code_industry.gitee.io/code_heavy_industry/pro002-maven/chapter10/verse01.html
- https://www.bilibili.com/video/BV12q4y147e4?p=162

## jar包冲突问题

### 类别

jar包冲突通常是以下两种情况导致：

1. **同一个jar包的不同版本。**在 Maven 的版本仲裁后，代码中使用了jar包中不存在的类。

   ![image-20220410113406457](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220410113406457.png)

2. **不同jar包中的同名类。**两个jar包中的包名和类名完全一样，但是其中的内容不同，会导致在使用时出错。拿 netty 来举个例子，netty 是一个类似 Tomcat 的 Servlet 容器。通常我们不会直接依赖它，所以基本上都是框架传递进来的。那么当我们用到的框架很多时，就会有不同的框架用不同的坐标导入 netty。大家可以参照下表对比一下两组坐标：

   | 截止到3.2.10.Final版本以前的坐标形式：                       | 从3.3.0.Final版本开始以后的坐标形式：                        |
   | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | <dependency> <br /><groupId>**org.jboss.netty**</groupId> <artifactId>**netty**</artifactId> <version>**3.2.10.Final**</version><br /> </dependency> | <dependency> <br /><groupId>**io.netty**</groupId> <artifactId>**netty**</artifactId> <version>**3.9.2.Final**</version> <br /></dependency> |

   但是偏偏这两个**『不同的包』**里面又有很多**『全限定名相同』**的类。例如：

   > org.jboss.netty.channel.socket.ServerSocketChannelConfig.class
   >
   > org.jboss.netty.channel.socket.nio.NioSocketChannelConfig.class 
   >
   > org.jboss.netty.util.internal.jzlib.Deflate.class 
   >
   > org.jboss.netty.handler.codec.serialization.ObjectDecoder.class 
   >
   > org.jboss.netty.util.internal.ConcurrentHashMap$HashIterator.class
   >
   >  org.jboss.netty.util.internal.jzlib.Tree.class 
   >
   > ……

### 解决方法

基本的解决思路为：

1. 先找到存在冲突的 jar 包；
2. 在冲突的 jar 包中选择一个，然后通过 exclusions 排除依赖，或是明确声明依赖。

我们可以使用 IDEA 和 Maven 提供的插件找到工程中存在的 jar 包冲突，两个插件的侧重点不同：

**IDEA 的 Maven Helper插件： **能够给我们罗列出来同一个 jar 包的不同版本，以及它们的来源。但是对不同 jar 包中同名的类没有办法。

**Maven 的 enforcer 插件：**既可以检测同一个 jar 包的不同版本，又可以检测不同 jar 包中同名的类。

在 POM.xml 文件中配置 enforcer 插件，执行 Maven 命令： `mvn clean package enforcer:enforce` 。

```xml
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-enforcer-plugin</artifactId>
                <version>1.4.1</version>
                <executions>
                    <execution>
                        <id>enforce-dependencies</id>
                        <phase>validate</phase>
                        <goals>
                            <goal>display-info</goal>
                            <goal>enforce</goal>
                        </goals>
                    </execution>
                </executions>
                <dependencies>
                    <dependency>
                        <groupId>org.codehaus.mojo</groupId>
                        <artifactId>extra-enforcer-rules</artifactId>
                        <version>1.0-beta-4</version>
                    </dependency>
                </dependencies>
                <configuration>
                    <rules>
                        <banDuplicateClasses>
                            <findAllDuplicates>true</findAllDuplicates>
                        </banDuplicateClasses>
                    </rules>
                </configuration>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```

## 体系外jar包的导入

对于他人给的一个 jar 包，我们可以使用 Mavan 的 install 的install-file 目标，将该 jar 包安装到 Maven 仓库。

```sh
mvn install:install-file -Dfile=[体系外 jar 包路径] \
-DgroupId=[给体系外 jar 包强行设定坐标] \
-DartifactId=[给体系外 jar 包强行设定坐标] \
-Dversion=1 \
-Dpackage=jar
```

> Windows 系统下使用 `^` 符号换行；Linux 系统用 `\`

## 项目导出 jar 包

### Maven 工程

配置 POM.xml 文件如下后，运行命令：`mvn package`。

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <useUniqueVersions>false</useUniqueVersions>
                            <classpathPrefix>lib/</classpathPrefix>
                            <mainClass>[主启动类包名 + 类名]</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

### 普通 Java 工程

如下图所示，common-java 是一个非 Maven 工程，只有一些 Java 类。

![image-20220410154455892](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220410154455892.png)

打包方式如下：

![image-20220410154706885](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220410154706885.png)

![image-20220410154917299](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220410154917299.png)

![image-20220410155214228](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220410155214228.png)

![image-20220410155237477](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220410155237477.png)

![image-20220410155313270](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220410155313270.png)

![image-20220410155346712](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220410155346712.png)

![image-20220410155420212](https://raw.githubusercontent.com/yanjundong/image-bed/main/images/image-20220410155420212.png)

## 参考

- http://heavy_code_industry.gitee.io/code_heavy_industry/pro002-maven/
- https://maven.apache.org/guides/index.html

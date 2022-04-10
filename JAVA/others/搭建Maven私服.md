# 第一节 搭建 Maven 私服：Nexus

## 1、Nexus 安装

### ①下载地址

小诀窍：使用迅雷下载比直接用浏览器下载快很多

https://download.sonatype.com/nexus/3/latest-unix.tar.gz

### ②上传、解压

上传到 Linux 系统，解压后即可使用，不需要安装。但是需要**注意**：必须提前安装 JDK。

> [root@x nexus-3.37.0-01]# ll
> 总用量 96
> drwxr-xr-x. 3 root root 4096 2月 13 17:33 bin
> drwxr-xr-x. 2 root root 4096 2月 13 17:33 deploy
> drwxr-xr-x. 7 root root 4096 2月 13 17:33 etc
> drwxr-xr-x. 5 root root 4096 2月 13 17:33 lib
> -rw-r--r--. 1 root root 651 11月 20 01:40 NOTICE.txt
> -rw-r--r--. 1 root root 17321 11月 20 01:40 OSS-LICENSE.txt
> -rw-r--r--. 1 root root 41954 11月 20 01:40 PRO-LICENSE.txt
> drwxr-xr-x. 3 root root 4096 2月 13 17:33 public
> drwxr-xr-x. 3 root root 4096 2月 13 17:33 replicator
> drwxr-xr-x. 23 root root 4096 2月 13 17:33 system

### ③启动 Nexus

> [root@x ~]# /opt/nexus-3.37.0-01/bin/**nexus start**
> WARNING: ************************************************************
> WARNING: Detected execution as "root" user. This is NOT recommended!
> WARNING: ************************************************************
> Starting nexus
> [root@x ~]# /opt/nexus-3.37.0-01/bin/**nexus status**
> WARNING: ************************************************************
> WARNING: Detected execution as "root" user. This is NOT recommended!
> WARNING: ************************************************************
> nexus is running.

### ④查看端口占用情况

> [root@x ~]# netstat -anp | grep java
> tcp 0 0 127.0.0.1:**45614** 0.0.0.0:* LISTEN 9872/java
> tcp 0 0 0.0.0.0:**8081** 0.0.0.0:* LISTEN 9872/java

上面 45614 这个每次都不一样，不用管它。我们要访问的是 8081 这个端口。但是需要**注意**：8081 端口的这个进程要在启动 /opt/nexus-3.37.0-01/bin/nexus 这个主体程序**一、两分钟**后才会启动，请耐心等待。

### ⑤访问 Nexus 首页

首页地址：http://[Linux 服务器地址]:8081/

初始化界面还是很酷的：

![images](http://heavy_code_industry.gitee.io/code_heavy_industry/assets/img/img001.612496a3.png)

## 2、初始设置

![images](http://heavy_code_industry.gitee.io/code_heavy_industry/assets/img/img002.e1ac8197.png)

![images](http://heavy_code_industry.gitee.io/code_heavy_industry/assets/img/img003.97a620db.png)

这里参考提示：

- 用户名：admin
- 密码：查看 /opt/sonatype-work/nexus3/admin.password 文件

> [root@hello ~]# cat /opt/sonatype-work/nexus3/admin.password
> ed5e96a8-67aa-4dca-9ee8-1930b1dd5415

所以登录信息输入如下：

![images](http://heavy_code_industry.gitee.io/code_heavy_industry/assets/img/img004.266b8a05.png)

继续执行初始化：

![images](http://heavy_code_industry.gitee.io/code_heavy_industry/assets/img/img005.4b81e5ab.png)

给 admin 用户指定新密码：

![images](http://heavy_code_industry.gitee.io/code_heavy_industry/assets/img/img006.43ebb0ac.png)

匿名登录，启用还是禁用？由于启用匿名登录后，后续操作比较简单，这里我们演示禁用匿名登录的操作方式：

![images](http://heavy_code_industry.gitee.io/code_heavy_industry/assets/img/img007.9291087d.png)

完成：

![images](data:image/png;base64,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)

## 3、对接 Nexus

### ①通过 Nexus 下载 jar 包

#### [1]了解 Nexus 上的各种仓库

![images](http://heavy_code_industry.gitee.io/code_heavy_industry/assets/img/img009.7f737ed7.png)

| 仓库类型 | 说明                                           |
| -------- | ---------------------------------------------- |
| proxy    | 某个远程仓库的代理                             |
| group    | 存放：通过 Nexus 获取的第三方 jar 包           |
| hosted   | 存放：本团队其他开发人员部署到 Nexus 的 jar 包 |

| 仓库名称        | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| maven-central   | Nexus 对 Maven 中央仓库的代理                                |
| maven-public    | Nexus 默认创建，供开发人员下载使用的组仓库                   |
| maven-releasse  | Nexus 默认创建，供开发人员部署自己 jar 包的宿主仓库 要求 releasse 版本 |
| maven-snapshots | Nexus 默认创建，供开发人员部署自己 jar 包的宿主仓库 要求 snapshots 版本 |

初始状态下，这几个仓库都没有内容：

![images](http://heavy_code_industry.gitee.io/code_heavy_industry/assets/img/img010.e3573d0b.png)

#### #_2-使用空的本地仓库)[2]使用空的本地仓库

![images](data:image/png;base64,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)

```xml
  <!-- 配置一个新的 Maven 本地仓库 -->
  <localRepository>D:/maven-repository-new</localRepository>
```

#### #_3-指定-nexus-服务器地址)[3]指定 Nexus 服务器地址

把我们原来配置阿里云仓库地址的 mirror 标签改成下面这样：

```xml
<mirror>
	<id>nexus-mine</id>
	<mirrorOf>central</mirrorOf>
	<name>Nexus mine</name>
	<url>http://192.168.198.100:8081/repository/maven-public/</url>
</mirror>
```

这里的 url 标签是这么来的：

![images](http://heavy_code_industry.gitee.io/code_heavy_industry/assets/img/img012.5a3b1f11.png)



![images](http://heavy_code_industry.gitee.io/code_heavy_industry/assets/img/img013.959ab72e.png)

把上图中看到的地址复制出来即可。如果我们在前面允许了匿名访问，到这里就够了。但如果我们禁用了匿名访问，那么接下来我们还要继续配置 settings.xml：

```xml
<server>
  <id>nexus-mine</id>
  <username>admin</username>
  <password>atguigu</password>
</server>
```

这里需要**格外注意**：server 标签内的 id 标签值必须和 mirror 标签中的 id 值一样。

#### #_4-效果)[4]效果

找一个用到框架的 Maven 工程，执行命令：

```sh
mvn clean compile
```

下载过程日志：

> Downloading from nexus-mine: http://192.168.198.100:8081/repository/maven-public/com/jayway/jsonpath/json-path/2.4.0/json-path-2.4.0.pom
> Downloaded from nexus-mine: http://192.168.198.100:8081/repository/maven-public/com/jayway/jsonpath/json-path/2.4.0/json-path-2.4.0.pom (2.6 kB at 110 kB/s)
> Downloading from nexus-mine: http://192.168.198.100:8081/repository/maven-public/net/minidev/json-smart/2.3/json-smart-2.3.pom
> Downloaded from nexus-mine: http://192.168.198.100:8081/repository/maven-public/net/minidev/json-smart/2.3/json-smart-2.3.pom (9.0 kB at 376 kB/s)
> Downloading from nexus-mine: http://192.168.198.100:8081/repository/maven-public/net/minidev/minidev-parent/2.3/minidev-parent-2.3.pom
> Downloaded from nexus-mine: http://192.168.198.100:8081/repository/maven-public/net/minidev/minidev-parent/2.3/minidev-parent-2.3.pom (8.5 kB at 404 kB/s)
> Downloading from nexus-mine: http://192.168.198.100:8081/repository/maven-public/net/minidev/accessors-smart/1.2/accessors-smart-1.2.pom
> Downloaded from nexus-mine: http://192.168.198.100:8081/repository/maven-public/net/minidev/accessors-smart/1.2/accessors-smart-1.2.pom (12 kB at 463 kB/s)

下载后，Nexus 服务器上就有了 jar 包：

![images](http://heavy_code_industry.gitee.io/code_heavy_industry/assets/img/img014.cc0e87c3.png)

### ②将 jar 包部署到 Nexus

#### [1]配置 Maven 工程

```xml
<distributionManagement>
    <snapshotRepository>
        <id>nexus-mine</id>
        <name>Nexus Snapshot</name>
        <url>http://192.168.198.100:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

这里 snapshotRepository 的 id 标签也必须和 settings.xml 中指定的 mirror 标签的 id 属性一致。

#### [2]执行部署命令

```sh
mvn deploy
```

1

> Uploading to nexus-mine: http://192.168.198.100:8081/repository/maven-snapshots/com/atguigu/demo/demo07-redis-data-provider/1.0-SNAPSHOT/maven-metadata.xml
> Uploaded to nexus-mine: http://192.168.198.100:8081/repository/maven-snapshots/com/atguigu/demo/demo07-redis-data-provider/1.0-SNAPSHOT/maven-metadata.xml (786 B at 19 kB/s)
> Uploading to nexus-mine: http://192.168.198.100:8081/repository/maven-snapshots/com/atguigu/demo/demo07-redis-data-provider/maven-metadata.xml
> Uploaded to nexus-mine: http://192.168.198.100:8081/repository/maven-snapshots/com/atguigu/demo/demo07-redis-data-provider/maven-metadata.xml (300 B at 6.5 kB/s)
> [INFO] ------------------------------------------------------------------------
> [INFO] Reactor Summary:
> [INFO]
> [INFO] demo-imperial-court-ms-show 1.0-SNAPSHOT ........... SUCCESS [ 1.875 s]
> [INFO] demo09-base-entity ................................. SUCCESS [ 21.883 s]
> [INFO] demo10-base-util ................................... SUCCESS [ 0.324 s]
> [INFO] demo08-base-api .................................... SUCCESS [ 1.171 s]
> [INFO] demo01-imperial-court-gateway ...................... SUCCESS [ 0.403 s]
> [INFO] demo02-user-auth-center ............................ SUCCESS [ 2.932 s]
> [INFO] demo03-emp-manager-center .......................... SUCCESS [ 0.312 s]
> [INFO] demo04-memorials-manager-center .................... SUCCESS [ 0.362 s]
> [INFO] demo05-working-manager-center ...................... SUCCESS [ 0.371 s]
> [INFO] demo06-mysql-data-provider ......................... SUCCESS [ 6.779 s]
> [INFO] demo07-redis-data-provider 1.0-SNAPSHOT ............ SUCCESS [ 0.273 s]

![images](http://heavy_code_industry.gitee.io/code_heavy_industry/assets/img/img015.b413af9d.png)

### ③引用别人部署的 jar 包

#### [1]提出问题

- 默认访问的 Nexus 仓库：maven-public
- 存放别人部署 jar 包的仓库：maven-snapshots

#### [2]配置 Maven 工程

```xml
<repositories>
    <repository>
        <id>nexus-mine</id>
        <name>nexus-mine</name>
        <url>http://192.168.198.100:8081/repository/maven-snapshots/</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
        <releases>
            <enabled>true</enabled>
        </releases>
    </repository>
</repositories>
```

## 4、修改仓库配置

举例：修改 maven-central 仓库代理的远程库地址

![images](http://heavy_code_industry.gitee.io/code_heavy_industry/assets/img/img128.714c1100.png)

![images](http://heavy_code_industry.gitee.io/code_heavy_industry/assets/img/img129.c33151a5.png)

![images](data:image/png;base64,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)
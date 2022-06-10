# 一、Docker基础

 ## 1、基础

### 1.1 为什么会出现

虚拟机太重了，Linux发展出了另一种虚拟化技术：Linux容器（`Linux Containers`, LXC）。

Linux容器不是模拟一个完整的操作系统，而是对进程进行隔离。容器与虚拟机不同，不需要捆绑一整套操作系统，只需要对软件工作所需的库资源和设置。系统因此变得高效轻量并可以保证部署在任意环境的软件都能始终如一的运行。

### 1.2 Docker与虚拟机的比较

- Docker容器很快，启动和停止都可以在秒级实现，而传统的虚拟机方式需要数分钟才可以；
- Docker容器需要的资源很少，基本不消耗额外的系统资源，而传统的虚拟机则需要单独分配独占的内存、磁盘等资源。
- Docker通过类似Git设计理念的操作来方便用户获取、分发和更新应用镜像，存储复用，增量更新；
- Docker通过Dockerfile支持灵活的自动化创建和部署机制，以提高工作效率，并标准化流程。

### 1.2 安装docker

```java
1、检查内核版本，必须是3.10及以上
uname -r
//如果低于的话，需要升级软件包及内核版本
yum update
2、安装docker
yum install docker
3、输入y确认安装
4、启动docker
[root@localhost ~]# systemctl start docker
[root@localhost ~]# docker -v
Docker version 1.12.6, build 3e8e77d/1.12.6
5、开机启动docker
[root@localhost ~]# systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
6、停止docker
systemctl stop docker
```

### 1.3 Docker的基本组成

- 镜像（image）：image文件可以看作是容器的模版。Docker会根据image文件生成容器。同一个image可以生成多个同时运行的容器实例。
- 容器（Container）：是用镜像创建的运行实例。
  - 它可以被启动、开始、停止、删除，每个容器都是相互隔离、保证安全的平台。
  - `可以把容器看做是一个简易版的 Linux 环境`（包括 root 用户权限、进程空间、用户空间 和 网络空间等）和运行在其中的应用程序。【Linux 中的很多命令和 docker 命令相似】
- 仓库（Repository）：集中存放 `镜像` 文件的场所。仓库和仓库注册服务器是有区别的。仓库注册器上往往存放着多个仓库，每个仓库又包含了多个镜像，每个镜像有不同的标签（tag）。仓库也分为公开仓库和私有仓库两种形式。

## 2、容器

### 2.1 新建并启动容器

```sh
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

<h4>OPTIONS 说明</h4>

```sh
--name "新名称"	为容器指定一个新的名称；
-d	后台运行容器，并返回容器ID，也即启动守护式容器；

-i	以交互模式运行容器，通常与 -t 同时使用；
-t	为容器重新分配一个伪输入终端，通常与 -i 同时使用；

-P	随机端口映射；
-p	指定端口映射，有以下四种形式：
	ip:hostPort: containerPort
	ip::containerPort
	hostPort:containerPort
	containerPort
```

当运行`docker run`命令来创建并启动容器后，Dcoker在后台运行的标准操作包括：

- 检查本地是否存在指定的镜像，不存在就从公有仓库下载；
- 利用镜像创建一个容器，并启动该容器；【等价于先执行了`docker create` 命令，再执行`docker start` 命令。】
-  分配一个文件系统给容器，并在只读的镜像层外面**挂载一层可读写层**；
-  从宿主机配置的网桥接口中桥接一个虚拟接口到容器中去；
-  从网桥的地址池配置一个IP地址给容器；【容器有自己的内部网络和IP地址，如下图所示。使用`docker inspect` 可以获取容器的具体信息。】
- 执行用户指定的应用程序；
- 执行完毕后容器被自动终止。

![image-20210124191106545](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210124191106545.png)

### 2.2 列出所有在运行的容器

```sh
docker ps [OPTIONS]
```

<h4>OPTIONS 说明</h4>

```sh
-a	列出当前所有正在运行的容器 + 历史上运行过的
-l	显示最近创建的容器
-n	显示最近 n 个创建的容器: docker ps -n 3
-q	静默模式，只显示容器ID
--no-trunc	不截断输出
```

### 2.3 退出容器

- exit：容器停止退出
- ctrl + P + Q：容器不停止退出

### 2.4 启动容器

 ```sh
docker start [容器名/ID ]
 ```

### 2.5 停止容器

```sh
docker stop [容器名/ID]
```

### 2.6 强制停止容器

```sh
docker kill [容器名/ID]
```

### 2.7 删除已停止的容器

```sh
docker rm [容器ID]
```

一次性删除多个容器：

```sh
#类似于SQL语句中的嵌套查询 
docker rm -f ${docker ps -a -q}

docker ps -a -q | xargs docker rm
```

### 2.8 查看容器日志

```sh
docker logs -f -t --tail [容器ID]
-t	是加入时间戳
-f	跟随最新的日志打印
--tail [数字]	显示最后多少条记录

支持的选项包括：
-details	打印详细信息
-f	持续保持输出
-since string	输出从某个时间开始的日志
-tail string 输出最近的若干日志
```

### 2.9 查看容器信息

#### 查看容器详情

查看某容器的具体信息，会以json格式返回包括容器Id、创建时间、路径、状态、镜像、配置等在内的各项信息：

```sh
docker inspect [容器ID]
```

#### 查看容器内进程

查看容器内进程可以使用`docker top`命令。这个命令类似于Linux系统中的top命令，会打印出容器内的进程信息，包括PID、用户、时间、命令等。

![image-20210124193815792](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210124193815792.png)

#### 查看统计信息

查看统计信息可以使用`docker stats`子命令，会显示CPU、内存、存储、网络等使用情况的统计信息。

```sh
docker stats 61fb7f1f8419
支持的参数：
	-a	输出所有容器统计信息，默认尽在运行中
	-format string	格式化输出信息
	-no-stream	不持续输出，默认会自动更新持续输出实时结果
	-no-trunc	不截断输出信息
```

![image-20210124193929655](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210124193929655.png)

### 2.10 进入正在运行的容器并以命令行交互

```sh
docker exec -it 容器ID /bin/bash
或者
docker attach 容器ID
```

> 上述两个区别：
>
> - exec：是在容器中打开新的终端，并且可以启动新的进程。
> - attach：直接进入容器启动命令的终端，不会启动新的进程。
>
> 使用attach命令有时候并不方便。当多个窗口同时attach到同一个容器的时候，所有窗口都会同步显示；当某个窗口因命令阻塞时，其他窗口也无法执行操作了。

### 2.11 容器内拷贝文件到主机上

cp命令支持在容器和主机之间复制文件。

```sh
docker cp 容器ID:容器内路径 目的主机路径
例子：docker cp 10b9a3588ab9:/tmp/yum.log	/root

docker cp 目的主机路径 容器ID:容器内路径
例子：docker cp aaa.txt 61fb7f1f8419:/tmp/

支持的参数有：
	-a	打包模式，复制文件会带有原始的uid/gid信息；
	-L	跟随软连接。当原路径为软连接时，默认只复制链接信息，使用该选项会复制链接的目标内容。
```

![image-20210124194706637](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210124194706637.png)

### 2.12 导入和导出容器

当需要把容器从一个系统迁移到另一个系统时，可以使用Docker提供的导入和导出功能。

#### 导出容器

导出容器是指，导出一个已经创建的容器到一个文件，不管此时这个容器是否处于运行状态。可以使用`docker export`命令。

```sh
docker export -o test_for_run.tar 61fb7f1f8419
```

之后，可将导出的tar文件传输到其他机器上，然后再通过导入命令导入到系统中，实现容器的迁移。

#### 导入容器

使用`docker import`命令导入变成镜像，

```sh
docker import test_for_run.tar - test/centos:2.0
```

> 既可以使用`docker load`命令来导入镜像存储文件到本地镜像库，也可以使用`docker import`命令来导入一个容器快照到本地镜像库。这两者的区别在于
>
> - 容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），
> - 镜像存储文件将保存完整记录，体积更大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。

### 2.13 容器互联

#### 概念

`容器互联`是一种让多个容器中的应用进行快速交互的方式。它会在源和接收容器之间创建连接关系，接收容器可以通过容器名快速访问到源容器，而不用指定具体的IP地址。

#### 实现

使用`--link` 参数可以让容器之间安全的进行交互。

使用之前创建的mysql01容器，现在创建一个新的容器，并把它连接到mysql01容器上：

```sh
docker run -it --name mysqllink --link mysql01:mysql01 centos:centos7

--link 的参数格式为 name:alias，name为要连接的容器的名称，alias是别名。
```

Docker通过两种方式为容器公开连接信息：

- 更新环境变量，使用 env 命令查看mysqllink容器的环境变量（如下图），其中MYSQL01开头的环境变量是提供mysqllink容器连接mysql01容器时使用的，前缀采用大写的连接别名。

  ![image-20210124221017889](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210124221017889.png)

- 更新 `/etc/hosts` 文件，查看mysqllink容器的hosts文件：

  ![image-20210124220737804](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210124220737804.png)

## 3、镜像

### 3.1 获取镜像

```sh
docker pull [name][:tag]
例如：docker pull ubuntu:18.04
```

> 如果不指定tag，默认会选择latest标签。但是一般来说，镜像的latest标签意味着该镜像的内容会跟踪最新版本的变更而变化，内容是不稳定的。因此，从稳定性上考虑应避免使用。

> 严格地讲，镜像的仓库名称中还应该添加仓库地址（即registry，注册服务器）作为前缀，如果没有前缀则使用默认仓库。例如执行`docker pull ubuntu:18.04` 命令相当于执行 `docker pull registry.hub.docker.com/ubuntu:18.04`。
>
> 如果不使用默认仓库，例如从网易蜂巢的镜像源来下载`ubuntu:18.04`镜像，可以使用`docker pull hub.c.163.com/public/ubuntu:18.04`。

### 3.2 查看镜像信息

**使用`history`命令查看镜像历史：**，具体是指组成镜像文件的多个层的创建信息。

![image-20210124175156101](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210124175156101.png)

**使用`inspect`命令获取镜像的详细信息，包括制作者、适应架构、各层的数字摘要等。**

![image-20210124174859842](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210124174859842.png)


### 3.3 删除和清理镜像

```sh
docker rmi [options] IMAGE[IMAGE...]
支持选项如下：
	-f	强制删除镜像，即使有容器依赖它
```

**清理镜像**

使用Docker一段时间后，系统中可能会遗留一些临时的镜像文件，以及一些没有被使用的镜像，可以通过`docker image prune`命令来进行清理。

```sh
支持的选项有：
	-a	删除所有无用镜像，不光是临时镜像
```

### 3.4 commit操作

`docker commit`  是把正在运行的一个容器生成一个新的镜像。

```sh
docker commit -m="提交的描述信息" -a="author" 容器ID 要创建的目标镜像名：[标签名]
	容器ID：正在运行的容器ID
	
例子：docker commit -m="custom mysql" -a="yz"565c5a9f4163 bzk/customMySQL:1.1
```

### 3.5 存出和载入镜像

#### 存出镜像

如果要导出镜像到本地文件，可以使用`docker save`命令。

例如，导出本地的`redis:4.0` 镜像到`redis_4.0.tar`中。

```sh
[root@xk1301the001 myDocker]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/redis     4.0                 191c4017dcdd        9 months ago        89.3 MB
[root@xk1301the001 myDocker]# docker save -o ./redis_4.0.tar redis:4.0
[root@xk1301the001 myDocker]# ls
redis_4.0.tar
```

![image-20210124180617152](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210124180617152.png)

#### 载入镜像

可以使用`docker load`将导出的tar文件再导入到本地镜像库。支持`-i`选项，从指定文件中读入镜像内容。

```sh
docker load -i ./redis_4.0.tar
```

### 3.6 上传镜像

可以使用`docker push` 命令把镜像上传到仓库，默认是上传到 Docker Hub官方仓库。

### 3.7 加载原理

Docker镜像实际上是由一层一层的文件系统组成，这种层级的文件系统称为`UnionFS`（联合文件系统）。

> `UnionFS`（联合文件系统）是一种分层、轻量级并且高性能的文件系统，它支持==对文件系统的修改作为一次提交来一层层的叠加==，同时可以将不同目录挂载到同一个虚拟文件系统下，Union文件系统是Docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像，可以制作出各种具体的应用镜像。
>
> 特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。

![image-20210123220129434](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210123220129434.png)

`boofs(boot file system)`主要包含`bootloader`和`kernel`, `bootloader`主要是引导加载`kernel`， Linux刚启动时会加载`boots`文件系统，==在Docker镜像的最底层是`bootfs`==。这一层与我们典型的`Linux/Unix`系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。
`rootfs (root file system)`，在boots之上。包含的就是典型Linux系统中的`/dev, /proc, bin, /etc`等标准目录和文件。roots就是各种不同的操作系统发行版，比如Ubuntu，Centos等等。

![image-20210123215757611](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210123215757611.png)

## 4、Docker容器数据卷

### 4.1 概念

`Docker` 容器产生的数据，如果不通过`docker commit` 生成新的镜像，使得数据成为镜像的一部分保存下来，那么当容器删除之后，数据自然也就都没有了。

为了能够保存数据在 `docker`中我们使用`卷` 。卷的设计目的就是数据的持久化，完全独立于容器的生命周期，因此 Docker 不会在容器删除时删除其挂载的数据卷。

特点：

-  `数据卷`可以在容器之间**共享和重用**，容器间传递数据将变得高效与方便；
- 对`数据卷`内数据的修改会**立马生效**，无论是容器内操作还是本地操作；
- 对`数据卷`的更新不会影响镜像，解耦开应用和数据；
- `数据卷的生命周期`一直持续到没有容器使用它为止。

### 4.3 通过命令添加容器卷

<h3>命令：</h3>

```sh
docker run -it -v /宿主机绝对路径目录:/容器内目录 镜像名
例如：docker run -it -v /root/dataVolume:/root/dataVolumeContainer centos
```

上面的例子中我们把宿主机root路径下的一个文件夹和容器中的一个文件夹捆绑在一起，在创建该镜像时就会在容器中创建对应的文件夹。如下图：

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ20200402-161930.png)

当然，我们也可以通过 `docker inspect 容器ID` 查看是否挂载成功。

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ20200402-162821.png)



<h3>容器和宿主机之间共享数据</h3>

我们先在 `宿主机` 中的 `dataVolume` 文件夹中创建 `test.txt` ，然后进入`容器`并打开`dataVolumeContainer` 文件夹，则可以发现该文件夹中也存在着 `test.txt` ，这样就实现了共享数据。

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ20200402-162143.png)

我们还可以在容器中通过 `vi test.txt` 命令修改 `test.txt` 文件，同时在`宿主机` 中就可以看到更改后的`test.txt` 。如下图：

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ20200402-162348.png)

<h3>容器停止退出后，宿主机修改了文件内容，重新启动容器时，容器中的文件能否同步？</h3>

我们可以先停止`该容器` ，然后修改`dataVolume` 中的 `test.txt` 文件。再启动该`centos容器`，并进入容器，可以发现`dataVolumeContainer` 中的内容也同步更改了。

<h3>命令（带只读权限）</h3>

```sh
docker run -it -v /宿主机绝对路径目录:/容器内目录:ro 镜像名
```

> 与普通的命令多了 `:ro`——>`read only`

### 4.4 DockerFile 添加容器卷

1. 根目录下新建 `mydocker` 文件夹并进入；

   ![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ20200406-202902.png)

2. 编辑`Dockerfile`文件，使用`VOLUME`指令来给镜像添加一个或多个数据卷；

3. File构建

   ```sh
   #volume test
   FROM centos
   VOLUME ["/dataVolumeContainer1", "/dataVolumeContainer2"]
   CMD echo "finished,--------------success1"
   CMD /bin/bash
   ```

4. build后生成镜像，生成一个新的镜像`yan/contos`；

   ```sh
   docker build -f /mydocker/Dockerfile -t yan/centos .
   ```

   > 注意最后的那个点

   ![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ20200406-203712.png)

5. run 创建容器。在容器中也可以找到卷目录。

   ```sh
   docker run -it yan/centos
   ```

   ![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ20200406-204413.png)

6. 上面已经看到了创建的2个卷目录，那么对应的主机目录地址在哪里呢？

   我们重新打开一个终端，使用命令 `docker inspect 容器ID`，可以查看到：

   ![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ20200406-204944.png)



> 验证：在容器的 `dataVolumeContainer1` 目录中创建`test.txt` 文件，然后打开对应的主机目录可以看到 `test.txt` 文件。
>
> ![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ20200406-205324.png)

### 4.5 数据卷容器

如果需要在==多个容器之间共享一些持续更新的数据==，最简单的方式是使用`数据卷容器`。数据卷容器也是一个容器，但是它的目的是专门提供数据卷给其他容器挂载。

创建一个数据卷容器dbcontainer，并在其中创建一个数据卷挂载到 /dbdata:

```sh
docker run -it -v /dbdata --name dbcontainer centos:centos7
```

![image-20210124203519963](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210124203519963.png)

其他的容器可以使用`--volumes-from` 来挂载dbcontainer容器中的数据卷，如下：

```sh
docker run -it --volumes-from dbcontainer --name db1 centos:centos7
```

此时，容器db1挂载了一个数据卷到相同的/dbdata目录，两个容器任何一方在该目录下的写入，其他容器都可以看到。

![image-20210124204300457](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210124204300457.png)

> 可以多次使用`--volumes-from`参数来从多个容器挂载多个数据卷，还可以从其他已经挂载了容器卷的容器来挂载数据卷。
>
> 使用`--volumes-from`参数所挂载数据卷的容器自身并不需要保持在运行状态。
>
> 使用数据卷容器可以让用户在容器之间自由地升级和移动数据卷。

## 5、DockerFile解析

### 5.1 概念

`DockerFile` 是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本。

构建的步骤：

1. 编写`DockerFile`文件；
2. 执行`docker build`生成镜像;
3. 执行`docker run`运行。

一般而言，`Dockerfile`主体内容分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

```sh
# Dockerfile文件的描述信息、作者等

# 基础镜像，该信息必须写在首行
FROM centos:centos7

# 镜像维护者的信息
MAINTAINER yz<yz@163.com>

# 镜像操作指令，每运行一条RUN指令，镜像就添加一层，并提交。
RUN yum -y install vim
RUN yum -y install net-tools
RUN echo "build successfully!!!"

# 在创建容器时执行的指令
EXPOSE 80
CMD /bin/bash
```



### 5.2 DockerFile解析过程

**基础内容：**

- 每条保留指令都必须为大写字母，且后面要跟随至少一个参数；
- 指令按照从上到下的顺序执行；
- `#` 表示注释；
- 每条指令都会创建一个新的镜像层，并对镜像进行提交。

`Docker`执行`DockerFile`的大致流程：

1. docker 从基础镜像运行一个容器；
2. 执行一条指令并对容器做出修改；
3. 执行类似 `docker commit` 的操作提交一个新的镜像层；
4. docker 再基于刚提交镜像运行一个新的容器；
5. 执行 `Dockerfile` 中的下一条指令知道所有的指令都执行完成。

### 5.3 Dockerfile保留字指令

- `FROM`: 基础镜像，当前新镜像是基于哪个镜像的；
- `MAINTAINER`: 镜像维护者的姓名和邮箱地址；
- `RUN`: 容器构建时需要进行的命令；
- `EXPOSE`: 当前容器对外暴露的端口号；
- `WORKDIR`: 指定在创建容器后，终端默认登录后进入的工作目录，一个落脚点；
- `ENV`: 用来在构建镜像过程中设置环境变量；
- `ADD`：将宿主机目录下的文件拷贝进镜像，并且ADD命令还会自动处理URL和解压 tar 压缩包；
- `COPY`：类似ADD，拷贝文件和目录到镜像中；`COPY src dest` 或`COPY ["src","dest"]`
- `VOLUME`：容器数据卷，用于数据保存和持久化工作；
- `CMD`：指定一个容器启动时要运行的命令。`DockerFile`中可以有多个 CMD 指令，但只会有最后一个生效，CMD 会被 `docker run` 之后的参数替换；
- `ENTRYPOINT`: 指定一个容器启动时要运行的命令。`ENTRYPOINT` 的目的和 CMD 一样，都是在指定容器启动程序及参数，但`ENTRYPOINT`不会被 `docker run` 之后的参数覆盖，
- `ONBUILD`: 当构建一个被继承的 Dockerfile 时运行命令，父镜像在被子镜像继承后，父镜像的 onbuild 被触发。

> - `DockerFile`中可以有多个`CMD` 指令，但只会有最后一个生效，`CMD` 会被 `docker run` 之后的参数替换。例如启动tomcat容器时命令为`docker run -it -p 8888:8080 tomcat ls -l` 。这样的话`DockerFile`文件中的CMD指令就不会执行，因此tomcat也就不能启动。
>
> -  `ENTRYPOINT` 在docker run之后的参数会被当做参数传递给 `ENTRYPOINT`，之后形成新的命令组合。

### 5.4 通过Dockerfile创建镜像

编写完成`Dockerfile`之后，可以通过`docker build`命令来创建镜像。

```sh
docker build -f /mydocker/Dockerfile -t yan/centos .
支持的参数：
	-f	指定Dockerfile文件路径
	-t	指定生成镜像的标签信息，可以重复使用多次为镜像一次添加多个名称
```

![image-20210125094430122.png](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210125094430122.png)

### 5.5 案例

#### Base镜像（scratch）

`DockerHub`中99%的镜像都是通过在base镜像中安装和配置需要的软件构建出来的。

#### 实践1—自定义镜像centos

默认的`centos` 镜像不支持`Vim`命令，也不支持`ifconfig`命令。

![image-20210124160904968](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210124160904968.png)

在此基础镜像上，编写`DockerFile`文件重新构建新的镜像，使其支持`vim`命令和`ifconfig`命令，并且初始工作目录为`/tmp`。

```sh
# DockerFile文件
FROM centos:centos7
MAINTAINER yz<yz@163.com>

ENV MYPATH /tmp
# 设置工作目录
WORKDIR ${MYPATH}

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo "build successfully!!!"
CMD /bin/bash
```

执行`docker build -f ./DockerFile -t yz/mycentos .` 命令开始构建，部分截图如下：

![image-20210124161803778](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210124161803778.png)

![image-20210124161827915](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210124161827915.png)

![image-20210124161735631](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210124161735631.png)

构建完成后，启动容器`mycentos` ，这个自定义镜像生成的容器可以使用`vim` 和`ifconfig` 命令。

![image-20210124161947909](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210124161947909.png)

![image-20210124162136026](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210124162136026.png)

### 5.6 最佳实践

- **精简镜像用途**：尽量让每个镜像的用途都比较集中单一，避免构造大而复杂、多功能的镜像；
- **选用合适的基础镜像**：容器的核心是应用。
- **减少镜像层数**：如果希望所生成镜像的层数尽量少，则要尽量合并RUN、ADD和COPY指令。通常情况下，多个RUN指令可以合并为一条RUN指令；
-  **提供注释和维护者信息**：`Dockerfile`也是一种代码，需要考虑方便后续的扩展和他人的使用；
- **及时删除临时文件和缓存文件**：特别是在执行apt-get指令后，`/var/cache/apt`下面会缓存了一些安装包；
- **提高生成速度**：如合理使用cache，减少内容目录下的文件，或使用`．dockerignore`文件指定等；

## 6、Docker安装

### MySQL

```bash
docker run -p 3306:3306 --name mysql-01 \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=root \
--restart=always \
-d mysql:5.7 
```



# Docker 实践

## 1、为镜像添加SSH服务

如何自行创建一个带有SSH服务的镜像以支持远程登录到容器。

### 1.1 基于 commit 命令

1. 下载`ubuntu` 镜像并启动一个容器；
2. 使用`apt` 安装并配置SSH服务；
3. 使用 `docker commit` 命令创建一个新的镜像；
4. 使用这个新的镜像启动容器，并添加端口映射`10011 -> 22` ，之后就可以通过宿主机的10011端口来登录容器。

### 1.2 基于 Dockerfile 创建

在`Dockerfile`文件中安装并配置SSH服务，然后使用 `docker build` 命令构建一个新的镜像，启动方式和上面的第4步一样。

## 2、SpringBoot工程的自动化部署

环境：

- `centos7.3`
- `docker 1.13.1`

### docker配置远程访问端口

```sh
vim /etc/sysconfig/docker
添加：OPTIONS='-Htcp://0.0.0.0:2375 -H unix:///var/run/docker.sock'
```

![image-20210304112921142](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210304112921142.png)

重启docker服务：

```sh
systemctl daemon-reload // 1，重新加载配置文件
systemctl restart docker // 2，重启docker
```

curl查看是否生效：

```sh
curl http://127.0.0.1:2375/version
```

![image-20210304152543117](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210304152543117.png)

### Idea连接docker

![image-20210304153051978](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210304153051978.png)

![image-20210304154115626](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210304154115626.png)

### 生成镜像并运行

编写`Dockerfile` 文件，存放路径可以为 `/src/resources/docker/` 下，文件为：

```sh
FROM java:8
VOLUME /tmp
#springboot-docker.jar为项目打包为jar包的名字，app.jar为别名
ADD springboot-docker.jar app.jar
#运行的时候对外提供的端口默认是8090，即便你在这里声明了3000也不会改变默认的端口8090
EXPOSE 8080
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

然后配置项目的 `pom.xml`，

```xml
<build>
    <finalName>springboot-docker</finalName>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <encoding>UTF-8</encoding>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        <!--使用docker-maven-plugin插件-->
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>docker-maven-plugin</artifactId>
            <version>1.0.0</version>
            <!--将插件绑定在某个phase执行-->
            <executions>
                <execution>
                    <id>build-image</id>
                    <!--将插件绑定在package这个phase上。也就是说，用户只需执行mvn package ，就会自动执行mvn docker:build-->
                    <phase>package</phase>
                    <goals>
                        <goal>build</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <!--指定生成的镜像名-->
                <imageName>yjd/${project.artifactId}</imageName>
                <!--指定标签-->
                <imageTags>
                    <imageTag>1.0</imageTag>
                </imageTags>
                <!--指定 Dockerfile 路径-->
                <dockerDirectory>${project.basedir}/src/main/resources/docker</dockerDirectory>
                <!--指定远程 docker api地址-->
                <dockerHost>http://192.168.1.149:2375</dockerHost>
                <!--这里是复制 jar 包到 docker 容器指定目录配置 -->
                <resources>
                    <resource>
                        <targetPath>/</targetPath>
                        <!--jar 包所在的路径  此处配置的 即对应 target 目录-->
                        <directory>${project.build.directory}</directory>
                        <!--需要包含的 jar包 ，这里对应的是 Dockerfile中添加的文件名　-->
                        <include>${project.build.finalName}.jar</include>
                    </resource>
                </resources>
            </configuration>
        </plugin>

    </plugins>
</build>
```

配置完成后，启动 `mvn package`，就可以将jar包打成docker镜像。运行的会很慢，在完成后会有新的镜像生成。

![image-20210304164903538](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210304164903538.png)

### 启动容器

![image-20210304165410193](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210304165410193.png)

![image-20210304165426014](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210304165426014.png)

### 一键同步代码

编辑Docker的配置：

![image-20210304170421456](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210304170421456.png)

添加maven的打包配置：

![image-20210304170307161](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210304170307161.png)

![image-20210304170348241](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210304170348241.png)

参考网上方法没有成功。








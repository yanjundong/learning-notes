# 一、Linux系统

## 1、系统分区

### 1.1 挂载

- 必须分区
  - / （根分区）
  - swap分区（交换分区，内存的2倍，但是不超过2GB）
- 推荐分区
  - /boot	 （启动分区，200MB）——保证Linux系统的启动。

### 1.2 文件系统结构

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ截图20200601111553.png)

### 1.3 Linux各目录的作用

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ截图20200607095034.png)

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ截图20200607095553.png)

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ截图20200607101654.png)

# 二、Linux常用命令

## 1、目录处理命令

### 1.1  ls

命令名称：ls

命令所在位置：/bin/ls

执行权限：所有用户

功能描述：显示目录文件

选项：-a 显示所有文件，包括隐藏文件；

​			-l 详细信息显示

​			-d 查看目录属性

​			-i 文件的节点

### 1.2  mkdir

命令名称：mkdir

命令英文原意：make directories

命令所在路径：/bin/mkdir

执行权限：所有用户

选项：-p 递归创建

备注：可以同时创建多个目录，用空格隔开就可以

### 1.3 cd

命令英文原意：change directory

### 1.4  pwd

命令名称：pwd

命令英文原意：print working directory

### 1.5  rmdir

命令名称：rmdir

命令英文原意：remove empty directories

功能描述：删除空目录，如果文件不为空，则不能删除

### 1.6  cp

命令名称：cp

命令英文原意：copy

命令所在位置：/bin/cp

执行权限：所有用户

功能描述：复制文件或命令

语法：cp -rp [原文件或目录] [目标目录]

​			-r 复制目录

​			-p 保留文件属性【包括权限和最后修改时间】

### 1.7 mv

命令名称：mv

命令英文原意：move

命令所在位置：/bin/mv

执行权限：所有用户

语法：mv [原文件或目录] [目标目录]

功能描述：剪切文件、改名

例子：改名：$ mv java python

### 1.8  rm

命令名称：rm

命令英文原意：remove

命令所在位置：/bin/rm

执行权限：所有用户

选项：-r 删除目录

​			-f 强制删除

## 2、文件处理命令

### 2.1 touch

执行权限：所有用户

功能描述：创建空文件

### 2.2 cat

执行权限：所有用户

功能描述：显示文件内容

​			-n：显示行号

### 2.3 tac

功能描述：显示文件内容（反向列示）

### 2.4 more

cat 只适用于显示短内容文件，如果是大文件的话就需要使用 more。

语法：more [文件名]

​			（空格）或f 翻页

​			（Enter） 换行

​				q或Q 退出

功能描述：分页显示文件内容

范例：$ more /etc/services

### 2.5 less

命令所在目录：/usr/bin/less

语法：less [文件名]

功能描述：分页显示文件内容（可以向上翻页）

范例：$ less /etc/services

备注：可以通过 `pagup`向上翻页。 `pagdn或 空格` 向下翻页。向上找一行可以 按 上箭头。

​			`/service` 表示搜索关键字 “service”，按 `n` 表示查找下一个。

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ截图20200610102257.png)

### 2.6 head

命令所在路径： /usr/bin/head

功能描述：显示文件前面几行

​			-n 指定行数

范例： $ head -n 20 /etc/services

### 2.7 tail

功能描述：显示文件后面几行

​			-n 指定行数

​			-f 动态显示文件末尾内容（如果有新的变化，也会即时更新在屏幕上）

用法和 head 一样

###  2.8 链接 ln

命令英文原意：link

语法：ln -s [原文件] [目标文件]

​			-s 创建软链接

功能描述：生成链接文件

软链接：

1. 相同于 windows 中的快捷方法
2. lrwxrwxrwx 其中的 l 就代表软链接；软链接的权限都为  rwxrwxrwx
3. 文件大小只是符号链接
4. /temp/issue.soft -> /etc/issue 箭头指向源文件

硬链接特征：

1. 拷贝 `cp -p` + 同步更新
2. 不能跨分区

## 3、权限管理命令

### 3.1 chmod

语法：

<img src="https://gitee.com/yanjundong97/picBed/raw/master/images/QQ截图20200610110908.png" style="zoom:80%;" />

语法解释：

​	一二行是不同的修改方式

​	ugoa：分别表示所有者、所属组、其他人和所有人

​	+-=：分别表示 加权限、减权限和直接赋予权限（清空以前的权限）

​	权限的数字表示：r ---- 4; w ---- 2; x ----1;

​		例如：rwxrw-r-- 的权限数字表示为：764

功能描述：改变文件或目录权限

范例：

​	$ chmod g+w testfile --赋予文件testfile所属组写权限

​	$ chmod -R 777 testdir --修改目录testfile 及其目录下文件为所有用户具有全部权限。



文件目录权限的不同

| 代表字符 | 权限     | 对文件的含义                             | 对目录的含义               |
| -------- | -------- | ---------------------------------------- | -------------------------- |
| r        | 读权限   | 可以查看文件内容 cat/less/more           | 可以列出目录中的内容 ls    |
| w        | 写权限   | 可以修改文件内容（但是不能删除文件） vim | 可以在目录中创建、删除文件 |
| x        | 执行权限 | 可以执行文件                             | 可以进入目录 cd            |



### 3.2 chown

命令英文原意：change file ownership

命令所在路径：/bin/chown

执行权限：所有用户

语法：chown [用户] [文件或目录]

功能描述：改变文件或目录的所有者

示例：$ chown abc def --改变文件def 的所有者为abc



### 3.3chgrp

命令英文原意：change file group ownership

命令所在路径：/bin/chgrp

执行权限：所有用户

语法：chown [用户组] [文件或目录]

功能描述：改变文件或目录的所属组

示例：$ chgrp brother def --改变文件def 的所属组为 brother

### 3.4 umask

命令英文原意：the user file-creation mask

命令所在路径：shell 内置命令

执行权限：所有用户

语法：umask [-S]

​			-S 以rwx形式显示新建文件缺省权限

功能描述：显示、设置文件的缺省权限

示例：$ umask -S

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ截图20200610154141.png)

## 4、文件搜索

### 4.1 find

命令所在路径：/bin/find

执行权限：所有用户

语法：find [搜索范围] [匹配条件]

功能描述：文件搜索



```shell
$ find /etc -name init	
--在目录 /etc 中查找文件 init
-iname 不区分大小写

$ find / -size +204800
在根目录下查找大于100MB的文件
+n 大于	-n 小于	n等于
其中这个n是计算出的数据块的大小，在 Linux中一个数据块大小为 512字节，即 0.5KB。
100MB = 102400KB = 204800 数据块

$ find /home -user liu
在根目录下查找所有者为liu的文件
-group 根据所属组查找

$ find /etc -cmin -5
在/etc 下查找5分钟内被修改过属性的文件和目录
命令中的数值同上 +n 大于	-n 小于	n等于
-amin 访问时间 access
-cmin 文件属性 change [文件属性：ls -l 看到的内容，包括文件所属组、所属者等]
-mmin 文件内容 modify

$ find /etc -size + 163840 -a -size 204800
在 /etc 下查找大于 80MB 小于 100MB的文件
 -a 两个条件同时满足
 -o 两个条件满足其中一个即可
 上面两个参数可以将其他的命令配合使用，不是仅用于这里。

$ find /etc -name inittab -exec ls -l {} \;
在 /etc 下查找 inittab 文件并显示其详细信息
-exec / -ok 命令；-ok会询问是否操作

-type 根据文件类型查找
	f 文件	d 目录	l软链接文件

-inum 根据i节点查找
对于一些很奇怪的文件名可以通过 `ls -i` 获得文件的 i节点，然后通过该命令删除
```



### 4.2 locate

命令所在路径：/usr/bin/locate

执行权限：所有用户

语法：locate 文件名

功能描述：在文件资料库中查找文件

范例：$ locate inittab

如果想要忽略大小写，需要加上 `-i`



locate 这个命令是建立在文件资料库上的搜索，所以如果文件刚刚添加到系统，可能会搜索不到，可以用 `updatedb` 更新文件资料库。需要注意的部分目录中的文件不会出现在文件资料库中，例如 temp目录

### 4.3 which

命令所在路径：/usr/bin/which

执行权限：所有用户

语法：which 命令

功能描述：搜索命令所在目录及别名信息

范例：$ which ls

```shell
[yz@localhost ~]$ whereis ls
ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz /usr/share/man/man1p/ls.1p.gz
```



使用 `whereis` 命令可以查看命令所在目录和命令的用户手册所在位置。

### 4.4 grep

 命令所在路径：/bin/grep

执行权限：所有用户

语法：grep -iv [指定子串] [文件]

功能描述：在文件中搜索子串匹配的行并输出

​		-i	不区分大小写

​		-v	排除指定字串

范例: $ grep mysql /root/install.log

```shell
$ grep ^# -v /root/install.log
# 排除该文件中所有以# 开头的行
```



## 5、帮助命令

### 5.1 man

命令所在路径：/usr/bin/man

执行权限：所有用户

语法：man [命令或配置文件]

功能描述：获得帮助信息

范例：$ man ls

​			查看ls命令的帮助信息（包括命令的功能和选项功能）

​			$ man services

​			查看配置文件services的帮助信息



可以使用 `whatis + 命令名称` 只查看命令的功能；使用 `apropos + 配置文件名称`  查看这个配置文件的简短信息。

```shell
[yz@localhost ~]$ whatis ifconfig
ifconfig (8)         - configure a network interface
[yz@localhost ~]$ apropos services
chkconfig (8)        - updates and queries runlevel information for system services
db_replicate (1)     - Provide replication services
```

### 5.2 help

命令所在路径：Shell内置命令

语法：help 命令

功能描述：获得Shell内置命令的帮助信息

范例：$ help umash

​			查看umask命令的帮助信息

`man` 命令不能查看到 Shell 内置命令的帮助信息

## 6、压缩解压命令

### 6.1 gzip

命令英文原意：GUN zip

命令所在路径：/bin/gzip

执行权限：所有用户

语法：gzip [文件]

功能描述：压缩文件

压缩后文件格式： .gz

### 6.2 gunzip

命令英文原意：GUN unzip

命令所在路径：/bin/gunzip

执行权限：所有用户

语法：gunzip [文件]

功能描述：解压缩 .gz的压缩文件

范例： $ gunzip yangmi.gz

>只能压缩文件，不能压缩目录。
>
>压缩完成之后，不保留原文件

### 6.3 压缩 tar

命令所在路径：/bin/tar

执行权限：所有用户

语法：tar 选项 [-zcf] [压缩后文件名] [目录]

​		-c 打包

​		-v 显示详细信息

​		-f 指定文件名

​		-z 打包同时压缩

功能描述：打包目录

压缩后文件格式： .tar.gz



### 6.4 解压 tar

tar命令 解压缩语法：

​	-x 解包

​	-v 显示详细信息

​	-f 指定解压文件

​	-z 解压缩

范例： $ tar -zxvf gaoyuanyuan.tar.gz

### 6.5 zip

命令所在路径：/usr/bin/zip

执行权限：所有用户

语法：zip 选项 [-r] [压缩后文件名] [文件或目录]

​		-r 压缩目录

功能描述：压缩文件或目录

压缩后文件格式： .zip

> 压缩后保留原文件

### 6.6 unzip

命令所在路径：/usr/bin/unzip

执行权限：所有用户

语法：unzip [文件]

功能描述：解压缩 .zip的压缩文件

范例： $ unzip test.zip

## 7、网络命令

### 7.1 write

语法：write <用户名>

功能描述：给用户发信息

### 7.2 wall

语法：wall <信息>

功能描述：广播信息

### 7.3 ping

语法：ping 选项  ip地址

​		-c 指定发送次数

功能描述：测试网络连通性

### 7.4 last

语法：last

功能描述：列出 `目前与过去` 登入系统的用户信息

### 7.5 lastlog

语法：lastlog 

功能描述：显示所有用户最后登录的时间

范例： $ lastlog

​			$ lastlog -u yz

### 7.6 traceroute

语法：traceroute

功能描述：显示数据包到主机间的路径

范例： $ traceroute www.baidu.com

### 7.7 netstat

语法：netstat [选项]

功能描述：显示网络相关信息

选项：

​		-t TCP协议

​		-u UDP协议

​		-l 监听

​		-r 路由

​		-n 显示IP地址和端口号

范例：

​	$ netstat -tlun 	查看本机监听的端口

​	$ netstat -an	查看本机所有的网络连接

​	netstat -rn 查看本机路由表

### 7.8 挂载

命令名称：mount

命令位置：/bin/mount

执行权限：所有用户

命令语法：mount [-t 文件系统] 设备文件名 挂载点

范例：$ mount /dev/sr0 /mnt/cdrom

## 8、关机重启命令

### 8.1 shutdown

shutdown [选项] 时间

选项：

​		-c 取消前一个关机命令

​		-h 关机

​		-r 重启

### 8.2 reboot

重启命令

### 8.3 logout

退出登录

## 9、Vim常用操作

### 9.1  Vim工作模式

![](https://gitee.com/yanjundong97/picBed/raw/master/images/QQ截图20200707151237.png)

### 9.2 插入命令

| 命令 | 作用                 |
| ---- | -------------------- |
| a    | 在光标所在字符后插入 |
| A    | 在光标所在行尾插入   |
| i    | 在光标所在字符前插入 |
| I    | 在光标所在行行首插入 |
| o    | 在光标下插入新行     |
| O    | 在光标上插入新行     |

### 9.3 定位命令

| 命令          | 作用       |
| ------------- | ---------- |
| :set nu       | 设置行号   |
| :set nonu     | 取消行号   |
| gg            | 到第一行   |
| G             | 到最后一行 |
| nG            | 到第n行    |
| :n（n为行号） | 到第n行    |

| 命令 | 作用     |
| ---- | -------- |
| $    | 移到行尾 |
| 0    | 移到行首 |

### 9.4 删除命令

| 命令    | 作用                         |
| ------- | ---------------------------- |
| x       | 删除光标所在处字符           |
| nx      | 删除光标所在处后n个字符      |
| dd      | 删除光标所在行，ndd删除n行   |
| dG      | 删除光标所在行到文件末尾内容 |
| D       | 删除光标所在处到行尾内容     |
| :n1,n2d | 删除指定范围的行             |

### 9.5 复制与粘贴命令

| 命令 | 作用                         |
| ---- | ---------------------------- |
| yy   | 复制光标所在的那一行         |
| nyy  | 复制当前行以下n行            |
| p,P  | 粘贴在当前光标所在行下或行上 |
| dd   | 剪切当前行                   |
| ndd  | 剪切当前行以下n行            |

### 9.6 替换和取消命令

| 命令       | 作用                              |
| ---------- | --------------------------------- |
| r          | 取代光标所在处字符                |
| R          | 从光标所在处开始替换字符，ESC结束 |
| u          | 取消上一步操作                    |
| [Ctrl] + r | 重做上一个操作                    |

### 9.7 功能图

![](https://gitee.com/yanjundong97/picBed/raw/master/images/vim功能图.png)

## 10、rpm

### 10.1 查询是否安装

```sh
$ rpm -q 包名 #查询包是否安装
$ rpm -qa #查询所有已经安装的rpm包
```

### 10.2 查询软件包详细信息

```sh
$ rpm -qi 包名 # 查看软件包详细信息
选项：
	-i 查询软件信息
	-p 查询为安装包信息
```

### 10.3 查询包中文件安装位置

```sh
$ rpm -ql 包名 
选项：
	-l 列表
	-p 查询未安装包信息
```

### 10.4 查询软件包的依赖包

```sh
$ rpm -qR 包名
选项
	-R 查询软件包的依赖性
	-p 查询未安装包的信息
```



## 11、yum在线管理rpm包

### 11.1 安装

```sh
$ yum -y install 包名
选项：
	install 安装
	-y 自动回答yes
```



#  三、shell

## 1、输入输出重定向

### 1.1 输出重定向

![image-20200829225107856](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200829225107856.png)

![image-20200829225208647](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200829225208647.png)

### 1.2 输入重定向

## 2、通配符和特殊符号

 ![image-20200831161109470](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200831161109470.png)

![image-20200831162338645](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200831162338645.png)

## 3、位置参数变量

![image-20200831205346214](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200831205346214.png)

## 4、预定义变量

![image-20200831210929029](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200831210929029.png)

## 5、接受键盘输入

![image-20200831211005428](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200831211005428.png)

## 6、变量测试与内容替换

![image-20200831213621030](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20200831213621030.png)

## 7、环境变量配置文件

# 四、Linux文件系统

## 1、文件系统特性

【每个 `inode` 和 `block` 都有编号】

- `superblock`：记录此 `filesystem` 的整体信息，包括 `inode/block` 的总量、使用量、剩余量，以及文件系统的格式和相关信息等。
- `inode`：记录文件的属性，一个文件占用一个inode，同时记录此文件的数据所在的block号码。
- `block`：实际记录文件的内容，若文件太大，会占用多个block。

如下图所示， 文件系统先格式化出 inode 与block 的区块， 假设某一个文件的属性与权限数据是放置到 inode 4 号【下图较小方格内】，而这个 inode 记录了文件数据的实际放置点为 2, 7, 13, 15 这四个 block 号码， 此时操
作系统就能够据此来排列磁盘的读取顺序， 可以一口气将四个 block 内容读出来！   

![image-20201125112230775](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201125112230775.png)

> U盘使用的文件系统一般为 `FAT 格式`。这种格式的文件系统并没有 inode 存在， 所以 FAT 没有办法将这个文件的所有 block 在一开始就读取出来。它的读取方式会是这样【在读取完一个block后才能知道下一个block的位置】：
>
> ![image-20201125112856331](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201125112856331.png)

**怎么找到inode4和block1的呢？？？**

目录树是由根目录开始读起，系统根据挂载点可以找到挂载点的inode号码，此时就能得到根目录的inode内容，并依据inode内容读取根目录的区块内的文件名数据，在一层层往下读到正确的文件的inode内容。

## 2、EXT2文件系统

文件系统一开始就把 inode 和 block 规划好了，除非重新格式化，否则，inode和block就固定不变了。

因为文件系统都高达数百GB，为了管理庞大的inode和block，EXT2在格式化时选择了划分多个区块群组的方法。

![image-20201125154334290](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201125154334290.png)

### Boot Sector

启动扇区，可以安装启动引导程序。

### Data Block

放置文件数据的位置。【在EXT2中支持的区块大小有1K、2K和4K三种】，不同的大小导致文件系统能够支持的最大磁盘容量和最大单一文件容量不同，具体如下：

| Block大小          | 1KB  | 2KB   | 4KB  |
| ------------------ | ---- | ----- | ---- |
| 最大单一文件限制   | 16GB | 256GB | 2TB  |
| 最大文件系统总容量 | 2TB  | 8TB   | 16TB |

每个区块最多只能放置一个文件的数据；如果文件大于区块大小，需要占用多个区块；如果文件小于区块，剩余容量就直接浪费了。

### Inode Table

inode记录的数据至少会有这些：

- 读写权限（read、write、excute）
- 拥有者和拥有组（owner、group）
- 文件大小
- 建立或者状态改变的时间（ctime）
- 最近读取时间（atime）
- 最近修改时间（mtime）
- 该文件真正内容的指向（pointer）

inode 还具有以下特点：

- 每个 inode 大小均固定为 `128B` (新的 ext4 与 xfs 可设定到 256B)；
- 每个文件都仅会占用一个 inode，因此文件系统能够建立的文件数量与inode的数量有关。

inode 中记录了文件内容所在的 block 编号，但是每个 block 非常小，一个大文件随便都需要几十万的 block。而一个 inode 大小有限，无法直接引用这么多 block 编号【记录一个数据块要使用4B】。因此引入了间接、双间接、三间接引用。间接引用让 inode 记录的引用 block 块记录引用信息。

![image-20201125161601733](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201125161601733.png)

### Superblock（超级区块）

超级区块是记录整个文件系统相关信息的地方。没有超级区块，就没有整个文件系统。它记录的信息有：

- 数据区块与inode总量；
- 未使用和已使用的inode与数据区块数量；
- 数据区块与inode的大小（block为1、2、4K，inode为128B或256B）；
- 文件系统的挂载时间、 最近一次写入数据的时间、 最近一次检验磁盘 （ fsck） 的时间等文件系统的相关信息；
- 一个有效位数值，若此文件系统已被挂载， 则有效位为 0 ；若未被挂载，则有效位为 1 。  

### block bitmap（区块对照表）

用来记录哪些区块已被使用，哪些未使用。

### inode bitmap（inode对照表）

记录使用与未使用的 inode 号码 。

## 3、与目录树的关系

在Linux下建立一个目录时，==文件系统会分配一个inode与至少一个区块给该目录==。

其中， inode 记录该目录的相关权限与属性， 并可记录分配到的那块 block 号码；而 block 则是记录在这个目录下的文件名与该文件名占用的 inode 号码数据。  

![image-20201125163305611](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201125163305611.png)

## 4、挂载点

每个文件系统都有独立的inode、区块、超级区块等信息，这个文件系统要能够链接到目录树才能被我们使用。==将文件系统与目录树结合的操作称为【挂载】==。挂载点一定是目录，该目录为进入该文件系统的入口。

## 5、简单操作

- df： 列出文件系统的整体磁盘使用量；

![image-20201125170838063](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201125170838063.png)

![image-20201125170934085](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201125170934085.png)

- du： 查看文件系统的磁盘使用量（ 常用在查看目录所占磁盘空间）  

![image-20201125171055340](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201125171055340.png)

## 6、硬链接与符号链接：ln

### 硬链接

多个文件名多应到同一个inode号码就是硬链接。也就是说，建立硬链接时 只是在某个目录下新增一条文件名然后链接到某个inode号码。

```sh
[root@xk1301the001 yd]# ln ./softwore/nacos-server-1.3.1.tar.gz . <==建立硬链接的命令
[root@xk1301the001 yd]# ls
module  mysql  nacos-server-1.3.1.tar.gz  redis  softwore
[root@xk1301the001 yd]# ll -i nacos-server-1.3.1.tar.gz ./softwore/nacos-server-1.3.1.tar.gz 
# inode号码          链接数
538809098 -rw-r--r-- 2 root root 74805643 Sep 28 16:59 nacos-server-1.3.1.tar.gz
538809098 -rw-r--r-- 2 root root 74805643 Sep 28 16:59 ./softwore/nacos-server-1.3.1.tar.gz
```

![image-20201125185859823](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201125185859823.png)

- 硬链接仅能在单一文件系统中进行
- 不能链接目录。

### 符号链接

就是创建一个独立的文件， 而这个文件会让数据的读取指向它链接的那个文件的文件名。

```sh
[root@xk1301the001 yd]# ln -s ./softwore/nacos-server-1.3.1.tar.gz .	<==建立软链接
[root@xk1301the001 yd]# ls
module  mysql  nacos-server-1.3.1.tar.gz  redis  softwore
[root@xk1301the001 yd]# ll -i nacos-server-1.3.1.tar.gz ./softwore/nacos-server-1.3.1.tar.gz 
1075803942 lrwxrwxrwx 1 root root       36 Nov 25 18:50 nacos-server-1.3.1.tar.gz -> ./softwore/nacos-server-1.3.1.tar.gz
 538809098 -rw-r--r-- 1 root root 74805643 Sep 28 16:59 ./softwore/nacos-server-1.3.1.tar.gz

```

![image-20201125185910224](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201125185910224.png)
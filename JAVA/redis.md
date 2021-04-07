# 一、Redis入门

## 1. 概述

Redis 是速度非常快的非关系型（NoSQL）内存键值数据库，可以存储键和五种不同类型的值之间的映射。

键的类型只能为字符串，值支持五种数据类型：字符串、列表、集合、散列表、有序集合。

==Redis 支持很多特性，例如将内存中的数据持久化到硬盘中，使用复制来扩展读性能，使用分片来扩展写性能。==

## 2. 安装

## 3. 测试性能

`redis-benchmark` 压力测试工具。

## 4. Redis是单线程的

对Redis是单线程的理解——`《Redis深度历险》P165`

Redis内部并不是只有一个主线程，还有几个异步线程专门处理一些耗时的操作，比如：删除大key（`unlink`）、`flush async`等。

> Redis是单线程的
>
> Redis是基于内存操作，CPU并不是Redis的瓶颈，Redis的瓶颈是机器的内存和网络带宽。
>
> Redis是基于C语言写的，能够提供 100000+ 的QPS，不必 Memecache差。
>
> Redis为什么单线程还这么快？首先多线程并不一定就比单线程快，因为会有CPU上下文的切换。

# 二、五大数据类型

![image-20201210170853578](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201210170853578.png)

## 1. String（字符串）

```sh
192.168.1.149:0>set hello world
"OK"

192.168.1.149:0>get hello
"world"

192.168.1.149:0>del hello
"1"

192.168.1.149:0>get hello
null
```

## 2. Lists（列表）

Redis的列表是链表而不是数组，所以它的索引定位很慢，时间复杂度为O(n)。而且使用的是 `双向链接`。

```sh
192.168.1.149:0>rpush mylist item1
"1"

192.168.1.149:0>rpush mylist item2
"2"

192.168.1.149:0>rpush mylist item3
"3"

192.168.1.149:0>rpush mylist item3
"4"

192.168.1.149:0>lrange mylist 0 -1
 1)  "item1"
 2)  "item2"
 3)  "item3"
 4)  "item3"
192.168.1.149:0>lpop mylist
"item1"

192.168.1.149:0>llen mylist
"3"
```

> Redis低层存储的不是一个简单的 `LinkedList` ，而是一个`快速链表（quicklist）` 的结构。
>
> 因为普通链表需要存储大量的pre和next，造成空间浪费，还会造成内存碎片化。`quicklist`就是把`ziplist`和普通链表结合起来使用。默认单个`ziplist` 的长度是8KB。
>
> ![image-20201207175603841](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201207175603841.png)

## 3. Sets（集合）

```sh
192.168.1.149:0>sadd myset item1
"1"

192.168.1.149:0>sadd myset item2
"1"

192.168.1.149:0>sadd myset item2
"0"

192.168.1.149:0>scard myset # 返回集合存储的myset的基数 (集合元素的数量).
"2"

192.168.1.149:0>smembers myset
 1)  "item2"
 2)  "item1"
192.168.1.149:0>sismember myset item3 # 返回成员 item3 是否是存储的集合 myset的成员.
"0"
```

## 4. Hashes（哈希）

Hash的实现结构类似于Java中的HashMap，都是`数组+链表` 的方法。不同之处在于：

- Redis的字典的值只能是字符串

- 而且它们`rehash` 的方式不一样。Java的HashMap在扩容之后，需要一次性全部`rehash`，Redis为了提高性能，不堵塞服务，采用了渐进式`rehash`的策略。

  渐进式rehash会在rehash时，保留新旧两个hash结构，查询时会同时查询两个hash结构，然后在后续的定时任务以及hash操作指令（来自客户端的hset和hdel指令）中，一点一点转移。当搬迁任务完成后，就会完全使用新的hash结构。

  ![image-20201207202514440](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201207202514440.png)

```sh
192.168.1.149:0>hset myhash name zhangsan
"1"

192.168.1.149:0>hset myhash age 18
"1"

192.168.1.149:0>hset myhash address china
"1"

192.168.1.149:0>hgetall myhash # 返回 key 指定的哈希集中所有的字段和值。
 1)  "name"
 2)  "zhangsan"
 3)  "age"
 4)  "18"
 5)  "address"
 6)  "china"
192.168.1.149:0>hget myhash age # 返回 key 指定的哈希集中该字段所关联的值
"18"
```

> 扩容、缩容的规则：
>
> - 当Redis没有进行 `bgsave`，且 `负载因子>1` 时进行扩容；
> - 当 `负载因子>5` 时，强制扩容；
> - 当 `负载因子<0.1` 时，进行缩容。
>
> 为什么缩容时不用考虑bgsave？

## 5. Zset（有序集合）

Zset类似于Java中SortedSet和HashMap的结合体，一方面要保证value的唯一性，另一方面，每个value可以赋值一个score，代表这个value的排序权重。内部实现用的是`跳跃列表`。

```sh
192.168.1.149:0>zadd myzset 98 li
"1"

192.168.1.149:0>zadd myzset 99 jiao
"1"

192.168.1.149:0>zadd myzset 80 zhang
"1"

192.168.1.149:0>zrange myzset 0 -1 withscores #返回存储在有序集合key中的指定范围的元素。 返回的元素可以认为是按得分从最低到最高排列。 如果得分相同，将按字典排序。
 1)  "zhang"
 2)  "80"
 3)  "li"
 4)  "98"
 5)  "jiao"
 6)  "99"
```

### 5.1 跳跃列表

# 三、其他的用法

## 1、Geospatial 地理位置

可以来实现“附近的人”、“附近的餐馆”等类似功能。

### 1.1 GeoHash算法

> GeoHash算法将二维的经纬度数据映射到一维的整数，这样所有的元素都挂载到一条线上。
>
> 在二维上距离近的点在一维上也近。

![image-20201208202238288](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201208202238288.png)

然后在此基础上继续切分，

![image-20201208202356652](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201208202356652.png)

继续切分下去，正方形会越来越小，精度会越来越高。

GeoHash算法一共有3步：

**首先将经纬度变为二进制**

比如这样一个点（39.923201, 116.390705）
 纬度的范围是（-90，90），其中间值为0。对于纬度39.923201，在区间（0，90）中，因此得到一个1；（0，90）区间的中间值为45度，纬度39.923201小于45，因此得到一个0，依次计算下去，即可得到纬度的二进制表示，如下表：

![image-20201208203107559](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201208203107559.png)

最后得到纬度的二进制表示为：

```
10111000110001111001
```

同理可以得到经度的二进制表示为：

```
11010010110001000100
```

**第2步，就是将经纬度合并。**

经度占偶数位，纬度占奇数位，注意，0也是偶数位。

```
11100 11101 00100 01111 00000 01101 01011 00001
```

**第3步，按照Base32进行编码**

Base32编码表的其中一种如下，是用0-9、b-z（去掉a, i, l, o）这32个字母进行编码。具体操作是先将上一步得到的合并后二进制转换为10进制数据，然后对应生成Base32码。需要注意的是，将5个二进制位转换成一个base32码。上例最终得到的值为：

```undefined
  wx4g0ec1
```

> GeoHash算法存在的问题：
>
> - 边缘问题
>
>   ![image-20201208203716415](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201208203716415.png)
>
>   如图，如果车在红点位置，区域内还有一个黄点。相邻区域内的绿点明显离红点更近。但因为黄点的编码和红点一样，最终找到的将是黄点。这就有问题了。
>
>   要解决这个问题，很简单，只要再查找周边8个区域内的点，看哪个离自己更近即可。
>
> - 曲线突变问题
>
>   在上面切分的图中，0111和1000两个编码非常相近，但它们的实际距离确很远。所以编码相近的两个单位，并不一定真实距离很近，这需要实际计算两个点的距离才行。

### 1.2 基本用法

在Redis里，经纬度使用52位的整数进行编码，放入zset里，zset的value是元素的key，score是GeoHash的52位整数值。通过zset的score排序就可以得到坐标附近的其他元素。

```sh
# 增加
192.168.1.149:0>geoadd company 116.48105 39.996794 juejin
"1"

192.168.1.149:0>geoadd company 116.514203 39.905409 ireader
"1"

192.168.1.149:0>geoadd company 116.489033 40.007669 meituan
"1"
#加入多个三元组
192.168.1.149:0>geoadd company 116.562108 39.787602 jd 116.334255 40.027400 xiaomi	
"2"
```

> Redis没有提供元素的删除指令，但因为Geo存储结构使用的是zset，所以可以使用zset相关的指令操作Geo数据。可以使用zrem指令删除元素。

```sh
# geodist 指令可以用来计算两个元素之间的距离
192.168.1.149:0>geodist company juejin ireader km
"10.5501"
```

```sh
# geopos 指令可以获取集合中任意元素的经纬度坐标，可以一次获取多个。
192.168.1.149:0>geopos company juejin
 1)    1)   "116.48104995489120483"
  2)   "39.99679348858259686"

192.168.1.149:0>geopos company ireader
 1)    1)   "116.5142020583152771"
  2)   "39.90540918662494363"
```

> 

```sh
# geohash 可以获取元素的经纬度编码字符串，上面已经提到，它是 base32 编码。
# 你可以使用这个编码值去 http://geohash.org/${hash}中进行直接定位，它是 geohash 的标准编码值。
192.168.1.149:0>geohash company ireader
 1)  "wx4g52e1ce0"
```

![image-20201208201148853](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201208201148853.png)

```sh
# georadiusbymember 指令是最为关键的指令，它可以用来查询指定元素附近的其它元素
# 范围 20 公里以内最多 3 个元素按距离正排，它不会排除自身
192.168.1.149:0>georadiusbymember company ireader 20 km count 3 asc
 1)  "ireader"
 2)  "juejin"
 3)  "meituan"
# 三个可选参数 withcoord withdist withhash 用来携带附加参数
192.168.1.149:0>georadiusbymember company ireader 20 km withcoord withdist withhash count 3 asc
 1)    1)   "ireader"
  2)   "0.0000"
  3)   "4069886008361398"
  4)      1)    "116.5142020583152771"
   2)    "39.90540918662494363"


 2)    1)   "juejin"
  2)   "10.5501"
  3)   "4069887154388167"
  4)      1)    "116.48104995489120483"
   2)    "39.99679348858259686"


 3)    1)   "meituan"
  2)   "11.5748"
  3)   "4069887179083478"
  4)      1)    "116.48903220891952515"
   2)    "40.00766997707732031"
```

```sh
# georadius 指令可以根据坐标值来查询附近元素，使用方法和 georadiusbymember 基本一致
192.168.1.149:0>georadius company 116.514202 39.905409 20 km withdist count 3 asc
 1)    1)   "ireader"
  2)   "0.0000"

 2)    1)   "juejin"
  2)   "10.5501"

 3)    1)   "meituan"
  2)   "11.5748"
```

## 2、Hyperloglog

优点：占用的内存固定，2^64 个不同元素的技术，只需要占用12KB的内存，如果要从占用内存的角度考虑，`Hyperloglog` 一定是首选。

**网页的 UV （一个人访问一个网站多次，但是还是算作一个人！）**

 传统的方式， set 保存用户的id，然后就可以统计 set 中的元素数量作为标准判断 !

 这个方式如果保存大量的用户id，就会比较麻烦！我们的目的是为了计数，而不是保存用户id；

该方法会有 0.81% 错误率！但对于 统计UV任务，可以忽略不计的！

```bash
127.0.0.1:6379> PFAdd hl1 a f i p e d g a # 创建第一组元素
(integer) 1
127.0.0.1:6379> PFCOUNT hl1 # 统计 hl1 中元素的基本数量
(integer) 7
127.0.0.1:6379> PFADD hl2 e a d g h d q l
(integer) 1
127.0.0.1:6379> PFMERGE hl3 hl1 hl2 # 取两组 hl1 和 hl2 的并集
OK
127.0.0.1:6379> PFCOUNT hl3
(integer) 10

```

### 2.1 使用方法

`Hyperloglog` 提供了两个指令`pfadd` 、`pfcount` ，来一个用户ID，就使用`pfadd`将用户ID塞进去。使用`pfcount`直接获取计数值。

### 2.2 实现原理

`《Redis深度历险》P42`

## 3、Bitmaps

> 位存储
>
> 使用：用户一年的签到记录，使用bitmaps存储，每天的签到记录只占用1个位。

Bitmaps 位图，数据结构，使用二进制来进行记录，只有0和1两种状态。

Redis的位数组是自动扩展的，如果设置了某个偏移位置超出了现有的内容范围，就会自动将数组进行零填充。

### 3.1 零存整取

![image-20201208095018568](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201208095018568.png)

```sh
192.168.1.149:0>setbit s 1 1
"0"

192.168.1.149:0>setbit s 2 1
"0"

192.168.1.149:0>setbit s 4 1
"0"

192.168.1.149:0>setbit s 9 1
"0"

192.168.1.149:0>setbit s 10 1
"0"

192.168.1.149:0>setbit s 13 1
"0"

192.168.1.149:0>setbit s 15 1
"0"

192.168.1.149:0>get s
"he"
```

除了`零存整取` 外，还可以 `零存零取`、 `整存零取`。

### 3.2 统计和查找

`bitcount` 指令用于统计指定范围内1的个数。

`bitpos` 指令用于查找指定范围内出现的第一个0或1。

==start和end参数是字节索引，即指定的范围必须是8的倍数。==

### 3.3 bitfield

对于位数组，如果要一次操作多位，可以使用功能强大的`bitfield`指令【也可以使用`管道`】。

`bitfield` 有3个子指令，分别是`get`、`set`、`incrby`，它们都可以对指定位片段进行读写，但是最多只能处理64位连续的位，如果超过，就要使用多个子指令。`bitfield`可以一次执行多个子指令。

> 有符号整型最大支持64位，而无符号整型最大支持63位。对无符号整型的限制，是由于当前Redis协议不能在响应消息中返回64位无符号整数。

[bitfield详细介绍](http://www.redis.cn/commands/bitfield.html)

## 4、布隆过滤器

布隆过滤器是一个不怎么精确的set结构，可以用来判断某个对象是否已经存在，但是会有一定的误差。

==当布隆过滤器说某个值存在时，这个值可能不存在；当它说某个值不存在时，它肯定不存在。==

> 新闻客户端的新闻浏览，如果要去除已经看过的内容，可以使用该结构。

### 4.1 基本用法

`bf.add` 和 `bf.exists` 这两个基本指令，可以用于添加元素和判断元素是否已经存在。如果要添加多个元素或者判断多个元素是否已经存在，可以使用`bf.madd` 和 `bf.mexists`。

Redis还提供了自定义参数的布隆过滤器，需要在add之前使用 `bf.reserve` 显示创建。`bf.reserve`提供了3个参数：

- error_rate：错误率。数值越低，需要的空间越大。默认值是0.01。
- initial_size：预计放入的元素数量。如果实际数量超过这个数值，错误率会上升。默认值是100。

### 4.2实现原理

![image-20201208174613726](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201208174613726.png)

布隆过滤器对应到Redis的数据结构里就是一个大型的位数组和几个不一样的无偏hash函数【无偏就是能够把元素的hash值算得比较均匀】。上图中的`f、g、h`就是这样的hash函数。

在向布隆过滤器中添加key时，使用多个hash函数对key进行hash，得到一个hash值，然后再对位数组长度取模，就会得到几个不同的位置，将这几位都置为1，就完成了add操作。

在查询某个元素是否存在时，也是同样的计算操作，然后比对这几个位置是否都是1，如果不是，则确定不存在；否则，这个元素极有可能存在。

在数组稀疏时，准确率很高。随着元素增多，数组变得稠密，准确率就会下降。

### 4.3空间占用估计

布隆过滤器有两个参数，第一个是预计元素的数量n，第二个是错误率f。有公式可以根据这两个输入得到两个输出：一个是位数组的长度l，一个是hash函数的最佳数量k。

```
k=0.7*(l/n) 	#约等于
f=0.6185^(l/n)	
```

## 5、漏斗限流

## 6、Scan

Redis中使用 `keys`指令可以列出满足特定正则表达式的key。但是这个指令存在2个很明显的缺陷：

- 没有offset、limit参数。
- 是遍历算法，时间复杂度是O(n)。如果实例中的key较多，会造成服务器卡顿，其他的指令必须处于等待状态。

`Scan`指令相比较`keys`的特点：

- 复杂度也是O(n)，但它是通过游标分步进行的，不会阻塞线程。
- 提供了limit参数。但是这个参数不是限制返回的条数，是指遍历的最大槽。
- 同`keys`一样，提供了模式匹配功能。
- 服务器不需要为游标保存状态，游标的唯一状态就是scan返回给客户端的游标整数。
- 返回的结果可能会有重复，需要客户端去重。
- 遍历的过程中如果有数据修改，改动后的数据能不能遍历到是不确定的。
- 单次返回的结果是空的不能说明遍历结束，而要看返回的游标是否为零。

### 6.1 字典的结构

在Redis中所有的key都存储在一个很大的字典中，类似于Java中的HashMap。

![image-20201208213332023](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201208213332023.png)

数组的大小总是2^n^，扩容一次数组，空间加倍。

### 6.2 scan遍历顺序

scan的遍历顺序不是按照从0位开始到末尾，而是采用了`高位进位加法`来遍历。这样的方法是考虑到字典的扩容和缩容时避免槽位的遍历重复和遗漏。

![image-20201208213842617](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201208213842617.png)



### 6.3 字典扩容

`《Redis深度历险》P83`

## 7、Info指令

Info指令显示的信息很多，分为9大块，分别是：

- `Server`：服务器运行的环境参数；
- `Clients`：客户端相关信息；
- `Memory`：服务器运行内存统计数据；
- `Persistence`：持久化信息；
- `Stats`：通用统计数据；
- `Replication`：主从复制相关信息；
- `CPU`：CPU使用情况；
- `Cluster`：集群信息；
- `KeySpace`：键值对统计数量信息。

```sh
192.168.1.149:0>info
"
# Server
redis_version:4.0.14
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:165c932261a105d7
redis_mode:standalone
os:Linux 3.10.0-957.el7.x86_64 x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:8.3.0
process_id:1
run_id:3a2db3d798e849d4c1895d7c58e2bbd6fe9ff283
tcp_port:6379
uptime_in_seconds:1662942
uptime_in_days:19
hz:10
lru_clock:13668501
executable:/data/redis-server
config_file:

# Clients
connected_clients:1 # Redis连接了多少客户端，可以使用 `client list` 指令查看来源
client_longest_output_list:0
client_biggest_input_buf:0
blocked_clients:0

# Memory
used_memory:848952	
used_memory_human:829.05K	# 内存分配器（jemalloc）从操作系统分配的内容容量
used_memory_rss:8368128
used_memory_rss_human:7.98M	# 操作系统看到的内存占用
used_memory_peak:849352
used_memory_peak_human:829.45K	# Redis内存消耗的峰值
used_memory_peak_perc:99.95%
used_memory_overhead:836278
used_memory_startup:786488
used_memory_dataset:12674
used_memory_dataset_perc:20.29%
total_system_memory:7882993664
total_system_memory_human:7.34G
used_memory_lua:37888
used_memory_lua_human:37.00K	# lua 脚本引擎占用的内存大小
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
mem_fragmentation_ratio:9.86
mem_allocator:jemalloc-4.0.3
active_defrag_running:0
lazyfree_pending_objects:0

# Persistence
loading:0
rdb_changes_since_last_save:0
rdb_bgsave_in_progress:0
rdb_last_save_time:1607432710
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:0
rdb_current_bgsave_time_sec:-1
rdb_last_cow_size:6578176
aof_enabled:0
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok
aof_last_cow_size:0

# Stats
total_connections_received:7
total_commands_processed:289
instantaneous_ops_per_sec:0 # 每秒执行多少次指令
total_net_input_bytes:8971
total_net_output_bytes:23162
instantaneous_input_kbps:0.00
instantaneous_output_kbps:0.00
rejected_connections:0	# 因为超出最大连接数限制而被拒绝的客户端连接次数
sync_full:0
sync_partial_ok:0
sync_partial_err:0	#半同步失败次数，如果太大就需要扩大复制积压缓冲区
expired_keys:0
expired_stale_perc:0.00
expired_time_cap_reached_count:0
evicted_keys:0
keyspace_hits:85
keyspace_misses:4
pubsub_channels:0
pubsub_patterns:0
latest_fork_usec:670
migrate_cached_sockets:0
slave_expires_tracked_keys:0
active_defrag_hits:0
active_defrag_misses:0
active_defrag_key_hits:0
active_defrag_key_misses:0

# Replication
role:master
connected_slaves:0
master_replid:84c7f9c6e178809599604b51bac23640d38b4dd7
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576	# 复制积压缓冲区大小，主节点会把修改指令先放在积压缓冲区，从节点会通过积压缓冲区来同步数据。因为该区域是环形的，如果有数据被覆盖，就需要进行全量同步模式【通过快照同步数据，十分耗费CPU和网络资源】。
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

# CPU
used_cpu_sys:1110.35
used_cpu_user:294.63
used_cpu_sys_children:0.02
used_cpu_user_children:0.00

# Cluster
cluster_enabled:0

# Keyspace
db0:keys=3,expires=0,avg_ttl=0
"
```

# 四、使用场景

## 1、计数器

可以对 String 进行自增自减运算，从而实现计数器功能。

Redis 这种内存型数据库的读写性能非常高，很适合存储频繁读写的计数量。

## 2、缓存

将热点数据放到内存中，设置内存的最大使用量以及淘汰策略来保证缓存的命中率。

## 3、会话缓存

使用 Redis 来统一存储多台应用服务器的会话信息。

当应用服务器不再存储用户的会话信息，也就不再具有状态，一个用户可以请求任意一个应用服务器，从而更容易实现高可用性以及可伸缩性。

## 4、分布式锁实现

在分布式场景下，无法使用单机环境下的锁来对多个节点上的进程进行同步。

可以使用 Redis 自带的 SETNX 命令实现分布式锁，除此之外，还可以使用官方提供的 `RedLock` 分布式锁实现。

## 5、其它

Set 可以实现交集、并集等操作，从而实现共同好友等功能。

ZSet 可以实现有序性操作，从而实现排行榜等功能。

# 五、Redis与Memcached

两者都是非关系型内存键值数据库，主要有 以下不同：

## 1、数据类型

Memcached 仅支持字符串类型，而 Redis 支持五种不同的数据类型，可以更灵活地解决问题。

## 2、数据持久化

Redis 支持两种持久化策略：RDB 快照和 AOF 日志，而 Memcached 不支持持久化。这也就意味着重启后Redis可以恢复数据，但Memcached 不可以。

## 3、分布式

Memcached 不支持分布式，只能通过在客户端使用一致性哈希来实现分布式存储，这种方式在存储和查询时都需要先在客户端计算一次数据所在的节点。

Redis Cluster 实现了分布式的支持。

## 4、内存管理机制

- 在 Redis 中，并不是所有数据都一直存储在内存中，可以将一些很久没用的 value 交换到磁盘【key仍然会在内存】，而 Memcached 的数据则会一直在内存中。
- Memcached 将内存分割成特定长度的块来存储数据，以完全解决内存碎片的问题。但是这种方式会使得内存的利用率不高，例如块的大小为 128 bytes，只存储 100 bytes 的数据，那么剩下的 28 bytes 就浪费掉了。

[Redis和Memcached的区别](https://honeypps.com/backend/difference-between-redis-and-memcached/)

# 六、事务

`Redis` 单条命令保证原子性，但是==事务不保证原子性==！

Redis事务本质：一组命令的集合，一个事务中的所有命令都会被序列化，在事务执行时，会按照顺序执行。

一次性、顺序性、排他性的执行一些命令。

==Redis事务没有隔离级别的概念！【因为Redis是单线程执行，不会有其他事务打搅】==

Redis的事务：

- 开始事务（`multi`）
- 命令入队（......）
- 执行事务（`exec`）

```bash
127.0.0.1:6379> multi # 开启事务
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> exec # 执行事务
1) OK
2) OK
3) "v2"
4) OK
```

> 放弃事务（DISCARD）

```bash
127.0.0.1:6379> multi # 开启事务
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> DISCARD # 取消事务
OK
127.0.0.1:6379> get k4 # 事务队列中命令都不会被执行！
(nil)
```

## 1、事务中的错误

- 事务在执行 EXEC 之前，入队的命令可能会出错。比如，`命令产生了语法错误`，`内存不足等`
- 事务在 EXEC 调用之后失败，比如事务中的命令处理了错误类型的键，将`字符串自增`等。

对于发生在 EXEC 执行之前的错误，服务器会对命令入队失败的情况进行记录，并在客户端调用 EXEC 时，拒绝执行并放弃该事务。

```bash
127.0.0.1:6379> multi 
OK
127.0.0.1:6379> set s1 "t1"
QUEUED
127.0.0.1:6379> set s2 "t2"
QUEUED
127.0.0.1:6379> sete s3 "t3" # 错误命令
(error) ERR unknown command 'sete'
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get s2 # 其他命令不会被执行
(nil)

```

至于在 EXEC 命令执行之后所产生的错误， 即使事务中有某个/某些命令在执行时产生了错误， 事务中的其他命令仍然会执行。

```bash
127.0.0.1:6379> set k1 "v1"
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> incr k1 # exec 执行后的错误
QUEUED
127.0.0.1:6379> set k2 "v2"
QUEUED
127.0.0.1:6379> set k3 "v3"
QUEUED
127.0.0.1:6379> exec
1) (error) ERR value is not an integer or out of range # exec 执行后的错误
2) OK
3) OK
127.0.0.1:6379> get k2 # 其他的语法依然会执行
"v2"

```

## 2、CAS实现乐观锁

`WATCH` 命令可以为Redis事务提供 ==check-and-set（CAS）==行为。

被监视的键 会发觉这些键是否被改动过，如果至少有一个被 WATCH 的键在 EXEC 执行之前被修改，整个事务都会被取消。

 ```bash
127.0.0.1:6379> set money 1000
OK
127.0.0.1:6379> watch money # 监视money
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> decrby money 100
QUEUED
127.0.0.1:6379> exec # 其他的客户端在执行 exec 之前，更改了money的值，事务执行失败。
(nil)

 ```

在事务执行失败后，程序需要做的，就是不断地重试这个操作，直到没有发生碰撞为止。

这种形式的锁称为 `乐观锁`，在大多数情况下，不同的客户端会访问不同的键，一般很少发生碰撞，所以重试的机会很少。

> 当 EXEC 命令被调用时，不管事务是否成功执行，对所有键的监视都会被取消。

# 七、Redis.conf

## 1、配置文件备注

## 2、INCLUDES

能够包含一个或者多个其他的配置文件。

```bash
# Include one or more other config files here.  This is useful if you
# have a standard template that goes to all Redis servers but also need
# to customize a few per-server settings.  Include files can include
# other files, so use this wisely.
#
# Notice option "include" won't be rewritten by command "CONFIG REWRITE"
# from admin or Redis Sentinel. Since Redis always uses the last processed
# line as value of a configuration directive, you'd better put includes
# at the beginning of this file to avoid overwriting config change at runtime.
#
# If instead you are interested in using includes to override configuration
# options, it is better to use include as the last line.
#
# include /path/to/local.conf
# include /path/to/other.conf
```

## 3、NETWORK 网络

如果没有 `bind` 的配置，Redis会监听所有IP的连接。当然，也可以指定监听一个或多个。

```bash
# By default, if no "bind" configuration directive is specified, Redis listens
# for connections from all the network interfaces available on the server.
# It is possible to listen to just one or multiple selected interfaces using
# the "bind" configuration directive, followed by one or more IP addresses.
#
# Examples:
#
# bind 192.168.1.100 10.0.0.1
# bind 127.0.0.1 ::1
#
# ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the
# internet, binding to all the interfaces is dangerous and will expose the
# instance to everybody on the internet. So by default we uncomment the
# following bind directive, that will force Redis to listen only into
# the IPv4 lookback interface address (this means Redis will be able to
# accept connections only from clients running into the same computer it
# is running).
#
# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
# JUST COMMENT THE FOLLOWING LINE.
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# bind 127.0.0.1
```



默认情况下的端口号为 6379，

```bash
# Accept connections on the specified port, default is 6379 (IANA #815344).
# If port 0 is specified Redis will not listen on a TCP socket.
port 6379
```

## 4、GENERAL

> 可以选择让Redis以守护进程的方式运行。（默认情况下 不是）

```conf
# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize no
```

> Redis pid的文件，如果是后台运行且没有指定文件名称，则默认为 `/var/run/redis.pid`

```bash
# If a pid file is specified, Redis writes it where specified at startup
# and removes it at exit.
#
# When the server runs non daemonized, no pid file is created if none is
# specified in the configuration. When the server is daemonized, the pid file
# is used even if not specified, defaulting to "/var/run/redis.pid".
#
# Creating a pid file is best effort: if Redis is not able to create it
# nothing bad happens, the server will start and run normally.
pidfile /var/run/redis_6379.pid
```

> 日志：设置日志级别。指定日志文件的名称。

```bash
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
loglevel notice

# Specify the log file name. Also the empty string can be used to force
# Redis to log on the standard output. Note that if you use standard
# output for logging but daemonize, logs will be sent to /dev/null
logfile ""
```

> 是否输出到系统日志

```bash
# To enable logging to the system logger, just set 'syslog-enabled' to yes,
# and optionally update the other syslog parameters to suit your needs.
# syslog-enabled no

# Specify the syslog identity.
# syslog-ident redis

# Specify the syslog facility. Must be USER or between LOCAL0-LOCAL7.
# syslog-facility local0
```

## 5、SNAPSHOTTING 快照

> 持久化到硬盘

```bash
#
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""

save 900 1	# 在900s内，如果有1个key改变，就保存
save 300 10	# 在300s内，如果有10个key改变，就保存
save 60 10000	# 在60s内，如果有10000个key改变，就保存
```

> 在备份错误时，是否停止写。默认是

```bash
# By default Redis will stop accepting writes if RDB snapshots are enabled
# (at least one save point) and the latest background save failed.
# This will make the user aware (in a hard way) that data is not persisting
# on disk properly, otherwise chances are that no one will notice and some
# disaster will happen.
#
# If the background saving process will start working again Redis will
# automatically allow writes again.
#
# However if you have setup your proper monitoring of the Redis server
# and persistence, you may want to disable this feature so that Redis will
# continue to work as usual even if there are problems with disk,
# permissions, and so forth.
stop-writes-on-bgsave-error yes
```

## 6、REPLICATION 复制

```bash
# Master-Slave replication. Use slaveof to make a Redis instance a copy of
# another Redis server. A few things to understand ASAP about Redis replication.
#
# 1) Redis replication is asynchronous, but you can configure a master to
#    stop accepting writes if it appears to be not connected with at least
#    a given number of slaves.
# 2) Redis slaves are able to perform a partial resynchronization with the
#    master if the replication link is lost for a relatively small amount of
#    time. You may want to configure the replication backlog size (see the next
#    sections of this file) with a sensible value depending on your needs.
# 3) Replication is automatic and does not need user intervention. After a
#    network partition slaves automatically try to reconnect to masters
#    and resynchronize with them.
#
# slaveof <masterip> <masterport>

# If the master is password protected (using the "requirepass" configuration
# directive below) it is possible to tell the slave to authenticate before
# starting the replication synchronization process, otherwise the master will
# refuse the slave request.
#
# masterauth <master-password> # 配置master节点密码，否则不能完成主从节点的同步
```

## 7、内存管理

> 设置内存使用限制为特定的比特数。

当内存达到上限时，Redis会根据移除策略（see maxmemory-policy）移除 key。

```bash
# Set a memory usage limit to the specified amount of bytes.
# When the memory limit is reached Redis will try to remove keys
# according to the eviction policy selected (see maxmemory-policy).
#
# If Redis can't remove keys according to the policy, or if the policy is
# set to 'noeviction', Redis will start to reply with errors to commands
# that would use more memory, like SET, LPUSH, and so on, and will continue
# to reply to read-only commands like GET.
#
# This option is usually useful when using Redis as an LRU or LFU cache, or to
# set a hard memory limit for an instance (using the 'noeviction' policy).
#
# WARNING: If you have slaves attached to an instance with maxmemory on,
# the size of the output buffers needed to feed the slaves are subtracted
# from the used memory count, so that network problems / resyncs will
# not trigger a loop where keys are evicted, and in turn the output
# buffer of slaves is full with DELs of keys evicted triggering the deletion
# of more keys, and so forth until the database is completely emptied.
#
# In short... if you have slaves attached it is suggested that you set a lower
# limit for maxmemory so that there is some free RAM on the system for slave
# output buffers (but this is not needed if the policy is 'noeviction').
#
# maxmemory <bytes>
```

> key的移除策略，默认情况下是永不过期`noeviction`。

```bash
# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
# is reached. You can select one from the following behaviors:
#
# volatile-lru -> Evict using approximated LRU, only keys with an expire set.
# allkeys-lru -> Evict any key using approximated LRU.
# volatile-lfu -> Evict using approximated LFU, only keys with an expire set.
# allkeys-lfu -> Evict any key using approximated LFU.
# volatile-random -> Remove a random key having an expire set.
# allkeys-random -> Remove a random key, any key.
# volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
# noeviction -> Don't evict anything, just return an error on write operations.
#
# LRU means Least Recently Used
# LFU means Least Frequently Used
#
# Both LRU, LFU and volatile-ttl are implemented using approximated
# randomized algorithms.
#
# Note: with any of the above policies, Redis will return an error on write
#       operations, when there are no suitable keys for eviction.
#
#       At the date of writing these commands are: set setnx setex append
#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
#       getset mset msetnx exec sort
#
# The default is:
#
# maxmemory-policy noeviction
```

![image-20201120163022213](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201120163022213.png)

> 作为内存数据库，出于对性能和内存消耗的考虑，Redis 实际使用的是一种近似LRU算法，并非针对所有 key淘汰，而是抽样一小部分并且从中选出被淘汰的 key。【随机采样出5个key（该数量可以设置），然后淘汰掉最旧的key。】
>
> LRU和LFU的实现 ====> `《Redis深度历险》P213`

## 8、AOF配置

> 默认情况下，是不开始AOF模式的，默认是使用RDB模式持久化的，

配置Redis持久化的 AOF 模式。

 # 八、Redis持久化

## 1. RDB（Redis DataBase）

在指定的时间间隔内将内存中的数据快照写入磁盘，也就是快照，在恢复时将快照文件直接读入内存里。

Redis 会单独创建（fork）一个子进程来进行持久化任务，会先把数据写入一个临时文件中，待持久化过程都结束，再用这个临时文件替换原来的持久化文件。

### fork的作用

Redis使用操作系统的多进程 `COW(Copy On Write)`机制来实现快照持久化。

快照持久化完全交给子进程来处理，父进程则继续处理客户端请求【这样意味着父进程会修改内存中的数据】。

在子进程刚刚创建后，它和父进程共享内存中的代码段和数据段。

![image-20201209102250560](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201209102250560.png)

当父进程对其中某一个页面的数据进行修改时，会先共享的页面复制出来一份，然后在这个复制的页面上进行修改。因为Redis实例中的冷数据占比较高，这也意味着被复制的页面只是一部分。

### rdb的保存文件

```bash
# The filename where to dump the DB
dbfilename dump.rdb # rdb保存的文件名

# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir ./ # rdb保存的路径
```

### 触发机制

1. save的规则满足的情况下

   ```bash
   save 900 1	# 在900s内，如果有1个key改变，就保存
   save 300 10	# 在300s内，如果有10个key改变，就保存
   save 60 10000	# 在60s内，如果有10000个key改变，就保存
   ```

2. 执行 `flushall` 命令，但产生的 `dump.rdb` 文件为空，没有意义。

3. 退出redis时

4. 在执行命令 `Save` 和 `bgsave` 后

   `save`：只管保存，其它不管，全部阻塞

   `bgsave`：异步进行快照操作，快照的同时还能响应客户端请求，可以通过 `lastsave` 命令查看最后一次成功执行快照的时间。

   ```bash
   127.0.0.1:6379> set k1 v1
   OK
   127.0.0.1:6379> bgsave # 执行该命令后，立即产生了 dump.rdb 文件
   Background saving started
   127.0.0.1:6379> lastsave 
   (integer) 1594690654
   
   ```

### 如何恢复rdb文件

1. 只需要把rdb文件放入 `dir`  指定目录，redis启动时就会自动检查 `dump.rdb`，恢复其中的数据。

### 优缺点

优点：

- 适合大规模的数据恢复
- 对数据的完整性要求不高

缺点：

- 需要一定的时间间隔，如果在这期间redis宕机了，会丢失数据
- fork进程时，会占用一定的内存空间

## 2. AOF（Append Only File） 

将所有的`写命令`都记录下来，恢复的时候把这个文件全部再执行一遍。

以日志的形式记录每个写操作，将 Redis 执行过的所有指令记录下来【先执行指令后记录】，只许追加文件，但不可以改写文件，redis启动之初会读取文件重新构建数据，也就是说，redis重启的话会根据日志文件的内容将写指令从前到后执行一遍以完成数据的恢复工作。

### aof的保存文件

```bash
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
#
# Please check http://redis.io/topics/persistence for more information.
 
appendonly no # 默认情况下是不开启的，需要手动改为yes

# The name of the append only file (default: "appendonly.aof")

appendfilename "appendonly.aof" # 保存的文件为 `appendonly.aof`

```

### appendonly.aof文件

![](https://img-blog.csdnimg.cn/20200714210522188.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDk3MTA1OQ==,size_16,color_FFFFFF,t_70)

### appendonly.aof文件修复

如果 `appendonly.aof` 这个文件有错误，这时 redis 不能启动，需要修改这个 aof 文件，

redis 提供了一个修复工具 `redis-check-aof`

```bash
redis-check-aof --fix appendonly.aof
```

### 重写规则

aof 规则默认就是文件的无限追加，文件会越来越大！为了避免出现这种情况，新增了重写机制。

当 AOF 文件的大小超过所设定的阀值时，Redis就会启动 AOF 文件的内容压缩，只保留可以恢复数据的最小指令集，可以使用命令 `bgrewriteaof`。

> 触发机制

Redis会记录上次重写时的 AOF 大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发。

```bash
# Automatic rewrite of the append only file.
# Redis is able to automatically rewrite the log file implicitly calling
# BGREWRITEAOF when the AOF log size grows by the specified percentage.
#
# This is how it works: Redis remembers the size of the AOF file after the
# latest rewrite (if no rewrite has happened since the restart, the size of
# the AOF at startup is used).
#
# This base size is compared to the current size. If the current size is
# bigger than the specified percentage, the rewrite is triggered. Also
# you need to specify a minimal size for the AOF file to be rewritten, this
# is useful to avoid rewriting the AOF file even if the percentage increase
# is reached but it is still pretty small.
#
# Specify a percentage of zero in order to disable the automatic AOF
# rewrite feature.

auto-aof-rewrite-percentage 100 # 设置重写的基准值
auto-aof-rewrite-min-size 64mb	# 设置重写的基准值
```

### 优缺点

优点： 

- 每次修改同步：`appendfsync always`   同步持久化 每次发生数据变更会被立即记录到磁盘  性能较差但数据完整性比较好。
- 每秒同步：`appendfsync everysec `   异步操作，每秒记录。如果一秒内宕机，有数据丢失。
- 不同步：`appendfsync no`  从不同步。

缺点：

- 相同数据集的数据而言aof文件要远大于rdb文件，恢复速度慢于rdb
- aof运行效率要慢于rdb,每秒同步策略效率较好，不同步效率和rdb相同

## 3. 选择

- 只做缓存：如果只希望数据在服务器运行的时候存在,你也可以不使用任何持久化方式.

- 同时开启两种持久化方式

  在这种情况下，当redis重启的时候会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整.

# 九、事件

Redis 服务器是一个事件驱动程序。

## 1、文件事件

服务器通过套接字与客户端或者其它服务器进行通信，文件事件就是对套接字操作的抽象。

Redis 基于 Reactor 模式开发了自己的网络事件处理器，使用 I/O 多路复用程序来同时监听多个套接字，并将到达的事件传送给文件事件分派器，分派器会根据套接字产生的事件类型调用相应的事件处理器。

![image-20201120164851836](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201120164851836.png)

## 2、时间事件

服务器有一些操作需要在给定的时间点执行，时间事件是对这类定时操作的抽象。

时间事件又分为：

- 定时事件：是让一段程序在指定的时间之内执行一次；
- 周期性事件：是让一段程序每隔指定时间就执行一次。

Redis 将所有时间事件都放在一个无序链表中，通过遍历整个链表查找出已到达的时间事件，并调用相应的事件处理器。

## 3、事件的调度与执行

服务器需要不断监听文件事件的套接字才能得到待处理的文件事件，但是不能一直监听，否则时间事件无法在规定的时间内执行，因此监听时间应该根据距离现在最近的时间事件来决定。

事件调度与执行由 aeProcessEvents 函数负责，伪代码如下：

```python
def aeProcessEvents():
    # 获取到达时间离当前时间最接近的时间事件
    time_event = aeSearchNearestTimer()
    # 计算最接近的时间事件距离到达还有多少毫秒
    remaind_ms = time_event.when - unix_ts_now()
    # 如果事件已到达，那么 remaind_ms 的值可能为负数，将它设为 0
    if remaind_ms < 0:
        remaind_ms = 0
    # 根据 remaind_ms 的值，创建 timeval
    timeval = create_timeval_with_ms(remaind_ms)
    # 阻塞并等待文件事件产生，最大阻塞时间由传入的 timeval 决定
    aeApiPoll(timeval)
    # 处理所有已产生的文件事件
    procesFileEvents()
    # 处理所有已到达的时间事件
    processTimeEvents()
```

将 aeProcessEvents 函数置于一个循环里面，加上初始化和清理函数，就构成了 Redis 服务器的主函数，伪代码如下：

```python
def main():
    # 初始化服务器
    init_server()
    # 一直处理事件，直到服务器关闭为止
    while server_is_not_shutdown():
        aeProcessEvents()
    # 服务器关闭，执行清理操作
    clean_server()
```

从事件处理的角度来看，服务器运行流程如下：

![image-20201120165133505](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201120165133505.png)

# 十、Redis的通信协议

## 1、RESP

Redis使用了浪费流量的文本协议`RESP(Redis Serialization Protocol)` 【因为Redis认为网络流量不是数据库系统的瓶颈】，它的优势在于：

- 实现过程异常简单
- 解析性能极好
- 人类可读

RESP将传输的结构数据分为5种最小单元类型，单元结束时统一加上回车换行符`\r\n`。

1. 单行字符串以 `+` 符号开头；
2. 多行字符串以 `$` 符号开头，后跟字符串长度；
3. 整数值以 `:` 符号开头，后跟整数的字符串形式；
4. 错误消息以 `-` 符号开头；
5. 数组以 `*` 符号开头，后跟数组的长度。

## 2、请求格式

请求格式只有一种，就是多行字符串数组。

`set key1 value1` 指令被序列化并输出后如下：

```sh
*3
$3
set
$5
key1
$7
value1
```

## 3、回复格式

单行字符串响应：

```tex
+OK\r\n
```

错误响应：

```tex
-Error message\r\n
```

整数响应：

```tex
:1000\r\n
```

多行字符串响应：

```tex
$6\r\nfoobar\r\n
```

数组响应：

```tex
*2\r\n$4\r\nname\r\n$5\r\nzhang\r\n
```



# 十一、Redis集群

主机数据更新以后，根据配置和策略，自动同步数据到备机的 `master/slave` 机制。`Master`以写为主，`Slave `以读为主。

## 1、一主二仆

复制了3份配置文件：

```bash
[root@hadoop104 bin]# cd redis_cluster/
[root@hadoop104 redis_cluster]# ll
total 0
drwxr-xr-x. 2 root root 24 Jul 14 16:17 6379
drwxr-xr-x. 2 root root 24 Jul 14 16:07 6380
drwxr-xr-x. 2 root root 24 Jul 14 16:18 6381
[root@hadoop104 redis_cluster]# cd 6379
[root@hadoop104 6379]# ll
total 60
-rw-r--r--. 1 root root 58783 Jul 14 16:17 redis.conf
[root@hadoop104 6379]# cd ../6380
[root@hadoop104 6380]# ll
total 60
-rw-r--r--. 1 root root 58783 Jul 14 16:07 redis.conf
[root@hadoop104 6380]# cd ../6381
[root@hadoop104 6381]# ll
total 60
-rw-r--r--. 1 root root 58783 Jul 14 16:18 redis.conf
```

更改 `redis.conf` 中的`port`、`pidfile`、`logfile`。

启动这3个服务：

```bash
[root@hadoop104 bin]# redis-server ./redis_cluster/6379/redis.conf 
[root@hadoop104 bin]# redis-server ./redis_cluster/6380/redis.conf 
[root@hadoop104 bin]# redis-server ./redis_cluster/6381/redis.conf 
[root@hadoop104 bin]# ps -ef|grep redis
root      17630      1  0 09:35 ?        00:00:16 redis-server *:6379
root      21952      1  0 16:19 ?        00:00:00 redis-server *:6380
root      21959      1  0 16:19 ?        00:00:00 redis-server *:6381
```

使用 `SLAVEOF 127.0.0.1 6379` 命令就可以实现 slave 节点跟master节点的同步。

```bash
127.0.0.1:6380> SLAVEOF 127.0.0.1 6379
OK
127.0.0.1:6380> info replication
# Replication
role:slave #该节点为slave节点
master_host:127.0.0.1
master_port:6379 
master_link_status:up #同步
master_last_io_seconds_ago:2
master_sync_in_progress:0
slave_repl_offset:0
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:ef4bdc917f8ac606f9ed6951e0b584ddad1df74b
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:0
```

> 在加入 slave 节点后，也会同步 master节点之前加入数据。

> 如果主节点down了，两个从节点会待命等待。

> 如果从节点down了，不会影响主节点，但是在重启之后，会成为另一个主节点，除非配置了 `redis.conf` 文件

## 2、薪火相传

上一个Slave可以是下一个slave的Master，Slave同样可以接收其他slaves的连接和同步请求，那么该slave作为了链条中下一个的master,可以有效减轻master的写压力。

`SLAVEOF 127.0.0.1 6380` 使6381节点成为 `6380` 的从节点。

```bash
127.0.0.1:6380> info replication
# Replication
role:slave	#6380节点是6379的从节点
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:7
master_sync_in_progress:0
slave_repl_offset:1204
slave_priority:100
slave_read_only:1
connected_slaves:1
slave0:ip=127.0.0.1,port=6381,state=online,offset=1204,lag=0	# 但也是6381节点的主节点
master_replid:ef4bdc917f8ac606f9ed6951e0b584ddad1df74b
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:1204
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:1204
```

## 3、反客为主

在主节点down之后，==手动==的把某一个从库变成主节点，停止与其他数据库的同步。

```bash
slaveof no one
```

## 4、复制原理

`《Redis深度历险》P124`

- slave启动成功连接到master后会发送一个sync命令
- Master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，
  在后台进程执行完毕之后，master将传送整个数据文件到slave,以完成一次完全同步
- 全量复制：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。
- 增量复制：Master继续将新的所有收集到的修改命令依次传给slave,完成同步。
- 但是只要是重新连接master,一次完全同步（全量复制)将被自动执行。

## 5、哨兵模式Sentinel 

### 作用

- **监控（Monitoring**）： 会不断地检查主节点和从节点是否运作正常。
- **提醒（Notification）**： 当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。
- **自动故障迁移（Automatic failover）**： 当一个主服务器不能正常工作时， `Sentinel` 会开始一次自动故障迁移操作， 它会将失效主服务器的其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器； 当客户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器。

### 配置、启动哨兵

自建一个`sentinel.conf`（名字不能改变） 文件在 `/usr/local/bin` （可以任意位置）。

在该文件中写入内容:

```bash
sentinel monitor 被监控数据库名字(自己起名字) 127.0.0.1 6379 1
#  上面最后一个数字1为 quorum 数，表示将该主节点判断为失效需要至少1个 sentinel 同意，如果sentinel 的数量不达标，就不会执行。
```

启动哨兵：

```bash
[root@hadoop104 bin]# redis-sentinel ./redis_cluster/sentinel.conf
22803:X 14 Jul 17:20:28.509 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
22803:X 14 Jul 17:20:28.509 # Redis version=4.0.9, bits=64, commit=00000000, modified=0, pid=22803, just
22803:X 14 Jul 17:20:28.509 # Configuration loaded
22803:X 14 Jul 17:20:28.510 * Increased maximum number of open files to 10032 (it was originally set to 
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 4.0.9 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in sentinel mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 26379
 |    `-._   `._    /     _.-'    |     PID: 22803
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

22803:X 14 Jul 17:20:28.511 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/s
22803:X 14 Jul 17:20:28.512 # Sentinel ID is bb27dd08ea01f8a0edfaa6e122d847f908153f3d
22803:X 14 Jul 17:20:28.512 # +monitor master redis:6379 127.0.0.1 6379 quorum 1
22803:X 14 Jul 17:20:28.512 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ redis:6379 127.0.0.1 6379
22803:X 14 Jul 17:20:28.513 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ redis:6379 127.0.0.1 6379

```

![image-20201209153320493](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201209153320493.png)

Redis Sentinel 集群可以看成是一个zookeeper集群，是集群高可用的心脏，一般由3~5个节点组成。

### 选举新的master

在把 `6379` 关闭后，哨兵就会开始选举新的master节点，选举的结果为 `6380`。`switch-master redis:6379 127.0.0.1 6379 127.0.0.1 6380`

```bash
22803:X 14 Jul 17:25:37.988 # +sdown master redis:6379 127.0.0.1 6379
22803:X 14 Jul 17:25:37.988 # +odown master redis:6379 127.0.0.1 6379 #quorum 1/1
22803:X 14 Jul 17:25:37.988 # +new-epoch 1
22803:X 14 Jul 17:25:37.988 # +try-failover master redis:6379 127.0.0.1 6379
22803:X 14 Jul 17:25:37.989 # +vote-for-leader bb27dd08ea01f8a0edfaa6e122d847f908153f3d 1
22803:X 14 Jul 17:25:37.989 # +elected-leader master redis:6379 127.0.0.1 6379
22803:X 14 Jul 17:25:37.989 # +failover-state-select-slave master redis:6379 127.0.0.1 6379
22803:X 14 Jul 17:25:38.080 # +selected-slave slave 127.0.0.1:6380 127.0.0.1 6380 @ redis:6379 127.0.0.1 6379
22803:X 14 Jul 17:25:38.080 * +failover-state-send-slaveof-noone slave 127.0.0.1:6380 127.0.0.1 6380 @ redis:6379 127.0.0.1 6379
22803:X 14 Jul 17:25:38.143 * +failover-state-wait-promotion slave 127.0.0.1:6380 127.0.0.1 6380 @ redis:6379 127.0.0.1 6379
22803:X 14 Jul 17:25:38.789 # +promoted-slave slave 127.0.0.1:6380 127.0.0.1 6380 @ redis:6379 127.0.0.1 6379
22803:X 14 Jul 17:25:38.789 # +failover-state-reconf-slaves master redis:6379 127.0.0.1 6379
22803:X 14 Jul 17:25:38.872 * +slave-reconf-sent slave 127.0.0.1:6381 127.0.0.1 6381 @ redis:6379 127.0.0.1 6379
22803:X 14 Jul 17:25:39.813 * +slave-reconf-inprog slave 127.0.0.1:6381 127.0.0.1 6381 @ redis:6379 127.0.0.1 6379
22803:X 14 Jul 17:25:39.813 * +slave-reconf-done slave 127.0.0.1:6381 127.0.0.1 6381 @ redis:6379 127.0.0.1 6379
22803:X 14 Jul 17:25:39.870 # +failover-end master redis:6379 127.0.0.1 6379
22803:X 14 Jul 17:25:39.870 # +switch-master redis:6379 127.0.0.1 6379 127.0.0.1 6380
22803:X 14 Jul 17:25:39.870 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ redis:6379 127.0.0.1 6380
22803:X 14 Jul 17:25:39.870 * +slave slave 127.0.0.1:6379 127.0.0.1 6379 @ redis:6379 127.0.0.1 6380
22803:X 14 Jul 17:26:09.928 # +sdown slave 127.0.0.1:6379 127.0.0.1 6379 @ redis:6379 127.0.0.1 6380

```

此时，集群自动调整为如下的结构：

![image-20201209154516373](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201209154516373.png)

> 在原来的master节点回来后，也会变成现在master节点的从节点。
>
> ```bash
> 22803:X 14 Jul 17:30:41.922 # -sdown slave 127.0.0.1:6379 127.0.0.1 6379 @ redis:6379 127.0.0.1 6380
> 22803:X 14 Jul 17:30:51.939 * +convert-to-slave slave 127.0.0.1:6379 127.0.0.1 6379 @ redis:6379 127.0.0.1 6380
> ```
>
> 此时，集群的结构如下：
>
> ![image-20201209154453773](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201209154453773.png)

### 选举规则

1. 删除列表中所有下线或者处于断线状态的从服务器。
2. 删除列表中所有最近5秒内没有回复过领头哨兵的从服务器。
3. 删除所有与已下线主服务器连接断开时间超过设置时间的从服务器，确保剩余服务器数据比较新。
4. 获取最高优先级中的复制偏移量最大的从服务器。
5. 如果还没有选出来，则按照ID排序，获取运行ID最小的从服务器。

## 6、Redis Cluster

`《Redis深度历险》P137`

# 十二、Redis缓存穿透和雪崩 

## 1、缓存穿透

`大量查询数据库中没有的数据`。因为缓存中获取不到，所有就会去查询数据源，比如用一个不存在的用户id查询用户信息。

解决办法：

- 布隆过滤器
- 对空结果也缓存

## 2、缓存击穿

`大并发请求一个key`。指一个key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在一个屏障上凿开了一个洞。

解决办法：

- 使用互斥锁
- 设置热点数据永不过期

## 3、缓存雪崩

`大量缓存集中失效`。

解决办法：

- 数据预热。把数据先预先访问一遍，

- 设置随机过期时间
- 设置热点数据永不过期

# 十三、Redis分布式锁

## 分布式锁的要求

要实现一个分布式锁，算法至少要满足以下3个要求：

- 安全性：独享。在任意一个时刻，只有一个客户端持有锁。
- 活性A：没有死锁。即便持有锁的客户端崩溃，锁仍然可以被获取。
- 活性B：容错性。只要大部分Redis节点都活着，客户端就可以获取和释放锁。

## 基于故障的转移存在的问题

使用Redis实现分布式锁最简单的方法就是在Redis中创建一个Key，这个Key存在一个失效时间【以保证最终能够释放锁】，当客户端释放资源时，删除这个key。

这里存在一个问题：

1. 单点失效问题，比如客户端A获得了锁，在master将数据同步到slave之前，master宕了，这时slave被晋升为master节点【数据没有被同步到这个节点】。此时客户端B又获得了这个锁【这个锁早已经被客户端A获得】。==安全失效==

![image-20201209173123361](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201209173123361.png)

## 单Redis实例的实现

暂时先不考虑上述问题的情况下，看下在单实例下怎么实现分布式锁。

获取锁使用命令:

```sh
SET resource_name my_random_value NX PX 30000
```

这个命令仅在不存在key的时候才能被执行成功（NX选项），并且这个key有一个30秒的自动失效时间（PX属性）。这个key的值是`my_random_value`(一个随机值），这个值在所有的客户端必须是唯一的，所有同一key的获取者（竞争者）这个值都不能一样。

value的值必须是随机数主要是为了更安全的释放锁，释放锁的时候使用脚本告诉Redis:只有key存在并且存储的值和我指定的值一样才能告诉我删除成功。可以通过以下Lua脚本实现：

```lua
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

> 使用这种方式释放锁可以避免误删别的客户端已经持有的锁。举个例子：客户端A取得资源锁，但是紧接着被一个其他操作阻塞了，当客户端A运行完毕其他操作后要释放锁时，原来的锁早已超时并且被Redis自动释放，并且在这期间资源锁又被客户端B再次获取到。如果仅使用DEL命令将key删除，那么这种情况就会把客户端B的锁给删除掉。

## RedLock算法

![image-20201124173338821](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20201124173338821.png)

### 失败时重试

当客户端无法取到锁时，应该在一个==随机延迟后重试==，防止多个客户端`同时`抢夺同一资源。

同样，客户端取得大部分Redis实例锁所花费的时间越短，脑裂出现的概率就会越低（必要的重试），所以，理想情况一下，客户端应该同时（并发地）向所有Redis发送SET命令。

> 当客户端从大多数Redis实例获取锁失败时，应该尽快地释放（部分）已经成功取到的锁。

### 缺陷
























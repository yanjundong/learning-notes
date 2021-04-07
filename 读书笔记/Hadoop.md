# HBase

## 1、客户端API

### 注意点

- 所有修改数据的操作都保证了==行级别的原子性==
- 创建`HTable`实例是有代价的，因为每个实例都需要扫描 `.META.` 表，以检查该表是否存在、是否可用，这些检查会导致实例的创建非常耗时。建议为执行的每一个线程创建独立的 `HTable`实例；
- 如果需要使用多个`HTable`实例，应该考虑使用 `HTablePool` 类，它为用户提供了一个复用多个实例的便捷方法。

### PUT

```java
Put put = new Put(rowKey);
put.add(HBaseOperator.FAMILY_NAME_BYTES, Bytes.toBytes(column), data);
table.put(put);
```

批量的添加操作如下，但使用该方法时，用户无法控制服务器执行 PUT 的顺序。：

```java
public void put(final List<Put> puts)
```

#### 1. 客户端的写缓冲区

每一个put操作实际上都是一个RPC操作（客户端把数据传送到服务器然后返回数据），这只适合小数据量的操作。HBase配备了一个客户端的写缓冲区，缓冲区负责收集put操作，然后调用RPC操作一次性将put送往服务器。以下是其方法：

```java
void setAutoFlush(boolean autoFlush, boolean clearBufferOnFail);
boolean isAutoFlush()
```

激活客户端缓冲区之后，单行的PUT操作不会产生RPC调用，存储的PUT实例会保存在客户端进程的内存中，当需要强制把数据写到服务端时，可以调用：

```java
void flushCommits() throws IOException;
```

客户端在把请求传到服务器之前，会按Region服务器排序分组，并通过每个Region服务器的RPC请求将数据传送到服务器。

![image-20210309161250400](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210309161250400.png)

配置客户端写缓冲区大小：

方法一：

```java
long getwriteBuffersize ()
void setwriteBufferSize (long writeBuffersize) throws IOException
```

方法二：

```xml
<property>
	<name>hbase.client.write.buffer</name>
    <value>20971520</value>
</property>
```

#### 2. 原子性操作CAS

```java
public boolean checkAndPut(final byte [] row,
      final byte [] family, final byte [] qualifier, final byte [] value,
      final Put put)
```

> 有一种特别的检查通过`checkAndPut()`调用来完成，即只有在另外一个值不存在的情况下，才执行这个修改。要执行这种操作只需要将参数value设置为null即可，只要指定列不存在，就可以成功执行修改操作。

### GET

```java
Get get = new Get(rowKey);
get.addColumn(HBaseOperator.FAMILY_NAME_BYTES, Bytes.toBytes(column));
Result result = table.get(get);
```

批量的获取方法如下：

```java
public Result[] get(List<Get> gets) throws IOException;
```

#### 1. Result类

当用户使用 `get()`方法获取数据时，会返回`Result`实例给用户。该类中提供了很多的方法可以获取数据，常用的有：

```java
public byte [] getRow();
public Cell[] rawCells();
public List<Cell> listCells();
public byte[] getValue(byte [] family, byte [] qualifier);
public boolean containsColumn(byte [] family, byte [] qualifier);
public NavigableMap<byte[], NavigableMap<byte[], NavigableMap<Long, byte[]>>> getMap();
public NavigableMap<byte[], NavigableMap<byte[], byte[]>> getNoVersionMap();
public NavigableMap<byte[], byte[]> getFamilyMap(byte [] family);
```

需要注意的是，无论使用什么方法获取Result中的数据，都不会产生额外的性能和资源消耗，因为这些数据都已经通过网络从服务器传输到了客户端。

### 批量处理操作

事实上，许多基于列表的操作，如 `get(List<Get> gets)` 和 `delete(final List<Delete> deletes)` 都是基于 `batch()` 方法实现的。

```java
public void batch(final List<?extends Row> actions, final Object[] results);
```

`Row` 类为 `Put`、`Get`、`Delete` 的父类，在该方法中可以放入这3中不同的操作。

> 需要注意的是，不可以将针对同一行的 Put 和 Delete 操作放在同一个批量处理请求中，因此这些操作的处理顺序可能会不同。

`public void batch(final List<?extends Row> actions, final Object[] results)` 和 `public Object[] batch(final List<? extends Row> actions)` 方法的区别是：

- 前一个可以让用户访问部分结果，而后者不行，因为结果数组在返回之前，控制流就中断了。

相同点是：

- get、put和delete都支持；
- 都不使用客户端缓冲区；

### 行锁

使用行锁可以保证只有一个客户端能获取一行数据相应的锁。

> 应该尽可能避免使用行锁，因为有可能会发生死锁，而死锁会浪费服务端的处理线程。如果必须使用，一定要节约占用锁的时间。

### 扫描

#### 1. 使用

使用 HTable的 `getScanner` 方法就可以返回真正的扫描器 `Scanner` 实例，用户可以使用它来迭代获取数据：

```java
ResultScanner getScanner(Scan scan) throws IOException;
ResultScanner getScanner(byte[] family) throws IOException;
ResultScanner getScanner(byte[] family, byte[] qualifier) throws IOException;
```

后两个方法已经隐式创建了一个 Scan 实例，逻辑中最后会调用 `getScanner(Scan scan)` 方法。

![image-20210309214459142](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210309214459142.png)

`Scan` 类的构造器有：

```java
Scan();
Scan(byte [] startRow);
Scan(byte [] startRow, Filter filter);
Scan(byte [] startRow, byte [] stopRow);
```

创建 Scan 实例后，还可以给它增加更多的限制条件：

```java
public Scan addFamily(byte [] family);
public Scan addColumn(byte [] family, byte [] qualifier);
```

因为 HBase 中的数据时按列族存储的，如果扫描不读取某个列族，那么该列族文件就不需要被读取，这就是列式存储架构的优势。

#### 2. ResultScanner类

扫描操作不会通过一次RPC请求返回所有匹配的行，而是以行为单位进行返回。`ResultScanner` 类会把每一行的数据封装为一个 Result 实例，并把所有的Result放入一个迭代器中，它的一些方法如下：

```java
Result next() throws IOException;
Result [] next(int nbRows) throws IOException;
void close();
```

> 应确保尽早释放扫描器实例，一个打开的扫描器会占用不少的服务器资源，累计多了会占用大量的堆空间。并且应该把 `close()` 方法放到 `try/finally` 中。

#### 3.缓存与批量处理

到目前，每一个 `next()` 调用都会为其生成一个单独的 RPC 请求，当单元格数据较小时，这样做的性能不会很好。可以使用**扫描器缓存**来一次RPC请求多行数据，默认情况下，这个缓存是关闭的。

可以在两个层面打开它：在表的层面，这个表所扫描实例的缓存都会生效：

```java
void setScannerCaching(int scannerCaching); 
int getScannerCaching();
```

也可以在扫描层面，这样便只会影响当前的扫描实例：

```java
void setCaching(int caching); // 设置缓存大小
int getCaching();	// 返回当前缓存大小
```



如果数据量是非常大的行，这些行有可能会超出客户端进程的内存容量，可以使用批量获取操作：

```java
void setCaching(int caching);
int getBatch();
```

批量是面向列一级的操作，可以选择每次 `ResultScanner` 实例返回多少列。

### Bytes类

`Bytes`类 支持大部分原生Java类型到字节数组的互转，另外还有些其它的方法如下：

![image-20210310104710300](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210310104710300.png)

### 过滤器

#### 1. 简介

过滤器可以在服务端生效，避免传输过多没用的数据给客户端。下图描述了过滤器怎么在客户端进行配置，并在网络传输中被序列化，然后被服务端执行。

![image-20210310110011856](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210310110011856.png)

#### 2. 比较过滤器

![image-20210310112154160](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210310112154160.png)

在创建比较过滤器时，需要一个比较运算符和比较器实例。每个比较过滤器的构造方法都有一个从 `CompareFilter` 继承来的签名方法：

```java
public CompareFilter(final CompareOp compareOp,
      final ByteArrayComparable comparator)
```

**比较运算符**包括：

![image-20210310111013857](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210310111013857.png)

**比较器**包括：

![image-20210310111103748](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210310111103748.png)

> 过滤器本来是为了筛掉无用的信息，但是所有基于 `CompareFilter` 的过滤器都正好相反，它们返回匹配的值。例如，如果需要返回大于或等于某值的数据，不应该使用 `LESS`，而应该使用 `GREATER_OR_EQUAL`。



- `RowFilter`：基于行键来过滤数据；
- `FamilyFilter`：使用列族名进行筛选；
- `QualifierFilter`：使用列名进行筛选；
- `ValueFilter` ：筛选出等于（或满足其他条件）指定值的单元格；
- `DependentColumnFilter` ：允许指定一个参考列，并使用参考列控制其他列的过滤；

#### 3. 专用过滤器

- `SingleColumnValueFilter`：用一列的值决定是否该行的数据被过滤；

  构造方法如下：

  ```java
  public SingleColumnValueFilter(final byte [] family, final byte [] qualifier,
        final CompareOp compareOp, final byte[] value);
  public SingleColumnValueFilter(final byte [] family, final byte [] qualifier,
        final CompareOp compareOp, final ByteArrayComparable comparator)
  ```

  还提供了一些辅助方法可以微调其过滤行为：

  ```java
  public void setFilterIfMissing(boolean filterIfMissing);
  public void setLatestVersionOnly(boolean latestVersionOnly);
  public boolean getFilterIfMissing();
  public boolean getLatestVersionOnly();
  ```

  

- `SingleColumnValueExcludeFilter`：

- `PrefixFilter`：所有与前缀匹配的行都会被返回客户端；

- `PageFilter`：可以对结果进行分页；

  > 因为不同的region服务器是分别工作的，它们之间不能共享状态和边界，因此，每个过滤器都会在完成扫描前获取 pageCount 行的结果，这种情况导致有可能返回的比所需要的多。

- `KeyOnlyFilter`：只获取key，而不需要value时可以使用；
- `InclusiveStopFilter`：该过滤器可以把结束行包括到结果中。
- `TimestampsFilter`
- `ColumnPrefixFilter`：列前缀过滤器，所以与设定前缀匹配的列都会包含在结果中；
- `RandomRowFilter`：随机行过滤器；

#### 4. 附加过滤器

一些额外的控制不依赖于这些过滤器本身，却可以应用到其他过滤器上，这就是 **附加过滤器** 想要提供的功能。

- `SkipFilter`：如果某一列检查没有通过，该过滤器会过滤掉整行的数据。因为`SkipFilter`需要通过`filterKeyValue()` 方法的返回结果决定如何处理该行，因此被包装的过滤器必须实现该方法。
- `WhileMatchFilter`：当有一条数据被过滤时，会直接放弃本次的扫描操作；

#### 5. FilterList

`FilterList` 允许用户使用多个过滤器共同限制返回的结果，它也实现了 `Filter` 接口。

```java
FilterList(final List<Filter> rowFilters);
FilterList(final Operator operator);
FilterList(final Operator operator, final List<Filter> rowFilters);
```

![image-20210310164821683](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210310164821683.png)

#### 6. 自定义过滤器

#### 7. 过滤器总结

![image-20210310165157185](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210310165157185.png)

### 计数器

#### 1. 单计数器

每次增加一列的计数器可以使用 `HTable` 类下的 `incrementColumnValue`方法。

```java
public long incrementColumnValue(final byte [] row, final byte [] family,
      final byte [] qualifier, final long amount);
public long incrementColumnValue(final byte [] row, final byte [] family,
      final byte [] qualifier, final long amount, final Durability durability);
```

#### 2. 多计数器

每次想要增加一行的多列，需要使用`HTable` 类下的 `increment`方法。

```java
Result increment(final Increment increment);
```

`Increment` 的构造方法如下，可以多次调用 `addColume` 方法来实现多计数器。

```java
Increment(byte [] row);
Increment addColumn(byte [] family, byte [] qualifier, long amount);
```

### 协处理器

### HConnection

`HTable` 实例的创建是一个非常耗时的操作，为客户端的每个请求创建一个实例是不现实的。应该在一开始创建实例，然后在客户端生命周期内不断重用。

但是，`HTable` 类不是线程安全的，本地的写缓冲区不能保证一致性。

通过通过 `HConnection` 类来解决该问题，它为 HBase 集群提供了客户端连接池，使用方法如下：

```java
HConnection hTablePool = HConnectionManager.createConnection(conf);
HTable table = hTablePool.getTable(tableName);
```

### 连接管理

`HTable` 实例共享同一个`HConnection` 实例时，提供的好处有：

- 共享 `ZooKeeper` 的连接：每个客户端需要与`ZooKeeper` 建立连接，查询表的`region`位置，这些信息可以在连接建立后缓存起来共享使用；

- 共享公共的资源：客户端需要通过`ZooKeeper`查找`-ROOT-`和`.META.`表，这个需要网络传输开销，客户端缓存这些公共资源后能够减少后续的网络传输开销，加快查找过程速度。

因此，与以下这种方式相比：

```
HTable table1 = new HTable("table1");
HTable table2 = new HTable("table2");
```

下面的方式更有效些：

```java
Configuration conf = HBaseConfiguration.create();
HTable table1 = new HTable(conf, "table1");
HTable table2 = new HTable(conf, "table2");
```



### 模式定义

#### 1. 表描述符

表描述符类`HTableDescriptor` 的构造函数如下：

```java
HTableDescriptor();	//空参数的构造器，仅仅只是为了在反序列化时创建一个空实例
HTableDescriptor(final TableName name);
HTableDescriptor(final HTableDescriptor desc);
```

`HTableDescriptor`  还提供了很多的 `getter` 和 `setter` 方法来设置表的其他属性，使用这些方法能微调表的性能。

**列族**

```java
void addFamily(final HColumnDescriptor family);
boolean hasFamily(final byte [] familyName);
HColumnDescriptor getFamily(final byte [] column);
HColumnDescriptor[] getColumnFamilies();
HColumnDescriptor removeFamily(final byte [] column);
```

**Region的大小设置**

```java
void setMaxFileSize(long maxFileSize);
```

`setMaxFileSize` 限制 Region存储的大小，如果存储文件超出了maxFileSize，会发生 Region 分裂。默认情况下，该值为 256MB。

**设置只读**

设置表为只读，默认情况下，表能够被修改。

```java
boolean isReadOnly();
void setReadOnly(final boolean readOnly);
```

**设置MemStore大小**

```java
void setMemStoreFlushSize(long memstoreFlushSize);
long getMemStoreFlushSize();
```

#### 2. 列族

列族描述符 `HColumnDescriptor`

**最大版本数**

HBase会移除超过最大版本数的数据，该参数默认值为3，如果不访问旧数据，也可以设置为1。

```java
int getMaxVersions();
HColumnDescriptor setMaxVersions(int maxVersions);
```

**块大小**

在HBase中，所有的存储文件都会划分为若干个小存储块，这些小存储块在 get或scan时会加载到内存中，这个参数的默认大小为 64KB。

```java
HColumnDescriptor setBlocksize(int s);
synchronized int getBlocksize();
```

这个参数用于指定 HBase 在一次读取过程中顺序读取多少数据到内存缓冲区。

**在内存中**

```java
HColumnDescriptor setInMemory(boolean inMemory);
```

该参数设置为 true 并不意味着会把整个列族的所有存储块都加载到内存，也不是说它们会长期保存在哪里，而是保证它们的高优先级。==在正常的数据读取过程中，块数据被加载到缓存区中并长期驻留在内存中，除非堆压力过大，这个时候才会强制从内存卸载这部分数据。==

该参数通常适合数据量较小的列族。

**布隆过滤器**

### HBaseAdmin

`HBaseAdmin` 提供了建表、创建列族、检查表是否存在、修改表结构和列族结构和删除表等功能。

#### 1. 表操作

```java
void createTable(HTableDescriptor desc);
// 下面2种方法可以在建表时进行预分区，将表在创建时就划分出若干特定的 region
void createTable(HTableDescriptor desc, byte [] startKey,
      byte [] endKey, int numRegions);
void createTable(final HTableDescriptor desc, byte [][] splitKeys);
```



```java
void createTableAsync(
    final HTableDescriptor desc, final byte [][] splitKeys)
```

上面的这个方法会进行**异步建表**，实际上 同步模式也仅仅是对异步模式的简单封装，通过不断地循环判断该任务是否已经完成。



在删除表之前，需要先确定该表是否已经被禁用：

```java
void disableTableAsync(final byte[] tableName);
void disableTable(final TableName tableName);
void disableTable(final byte[] tableName);
void disableTable(final String tableName);
```

在把表设置为禁用时，Region 服务器会将内存中近期还未提交的已修改的数据刷写到磁盘中，然后关闭所有的 region，并更新这张表的元数据，将所有的region标记为下线状态。

#### 2. 模式操作

```java
void modifyTable(final TableName tableName, final HTableDescriptor htd);
void addColumn(final byte[] tableName, HColumnDescriptor column);
void modifyColumn(final String tableName, HColumnDescriptor descriptor);
void deleteColumn(final byte[] tableName, final String columnName);
```

#### 3. 集群管理

`HBaseAdmin`类还允许用户查看集群当前的状态、执行表级人物和管理Region。



#### 4. 集群状态管理

调用 `HBaseAdmin#getClusterStatus()` 可以查询到 `ClusterStatus` 实例，该实例包含了 master 搜集到的整个集群的信息。

![image-20210311195742940](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210311195742940.png)



## 2、架构

### 数据的查找和传输

#### 1. B+树

- 效果远好于二叉树的划分，大大减少了查询特定主键所需要的IO操作；
- 可以提供高效的范围扫描功能【叶节点相互连接并且按主键有序】；

#### 2. LSM树

- 写入操作：输入数据会先存储在日志文件中防止数据丢失，然后把数据写入内存中即可；
- 删除/更新操作：都只需要在内存中插入一个删除/更新标记，真正的删除/更新操作会在合并过程中进行；
- 查询操作：相比较来说，查询操作是最费时的，需要依次查询`memtable`、`immutable memtable`、`SSTable0`、`SSTable1`......。因为后者的数据比前者的数据旧，所以一旦匹配到了需要的数据，就可以返回。【可以使用**布隆过滤器**或**索引文件**来加速读取】

**合并操作：**

LSM树中合并操作是最重要的操作，主要有两个作用：

1. 在内存数据满了后，合并内存中的数据到磁盘中；
2. 因为删除/更新操作只是插入标记，可以通过合并减少冗余数据；

目前广泛使用的有2中合并策略，`size-tiered`策略和`leveled`策略：

- `size-tiered`策略：是HBase采用的合并策略，具体内容是当某个规模的集合达到一定的数量时，将这些集合合并为一个大的集合。比如有5个50个数据的集合，那么就将他们合并为一个250个数据的集合。这种策略有一个缺点是当集合达到一定的数据量后，合并操作会变得十分的耗时。
- `leveled`策略：是LevelDB和RocksDB采用的合并策略，将数据划分为不同的层次，每一层的集合大小都是有限制的，并随着level层次的递增而递增，当某一层的数据达到上限时，会从该层选择一个文件与下一层合并；

### 存储

![image-20210222123815447](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210222123815447.png)

1）ZK

HBase 通过 Zookeeper 来做 Master 的高可用、RegionServer 的监控、元数据的入口以及 集群配置的维护等工作。

2）HMaster

Master 是所有 `Region Server` 的管理者，其实现类为 `HMaster`，主要作用如下：

- 对于表的操作：create, delete, alter（DML）
- 对于 RegionServer的操作：分配 regions到每个RegionServer，监控每个 RegionServer

3）HRegionServer

Region Server 为Region 的管理者，其实现类为 `HRegionServer` ，主要作用为：

- 对于数据的操作：get、put、delete（DDL）
- 对于Region的操作：分裂和合并Region

4）Region

一个Region对应HBase表的一个子表，当表太大（太小）时会发生分裂（合并）

5）Store

一个Store对应HBase表的一个列族，Store的存储有两种：MemStore和StoreFile。

6）StoreFile

保存实际数据的物理文件，StoreFile以 HFile的形式存储在HDFS上。

每个Store会有一个或多个StoreFile（HFile），数据在每个 StoreFile 中都是有序的。又因为在flush（如全局MemStore达到一定数后）后HFile文件的大小不同，所以还会涉及HFile的合并，在合并文件太大时还会切开。

7）MemStore

写缓存，由于 HFile 中的数据要求是有序的，所以数据是先存储在 MemStore 中，排好序后，等到达刷写时机才会刷写到 HFile，每次刷写都会形成一个新的 HFile。 

8）WAL(HLog)

为了防止掉电后MemStore中的数据丢失，数据会先写在一个叫做 Write-Ahead logfile 的文件 中，然后再写入 MemStore 中。所以在系统出现故障的时候，数据可以通过这个日志文件重 建。

#### HFile格式

![image-20210319113804051](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210319113804051.png)

- `Index块`记录Data和Meta块的偏移量；

- `Trailer`是持久化数据到文件结束时写入的，写入后即确定其成为不可变的数据存储文件，它有指向其他块的指针；

- `Data块` 的大小默认值为64KB；

- HDFS的默认块大小为128MB，对于HFile来说，它的一个Data块为64KB，这两者之间没有相关性；

  ![image-20210319165243674](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210319165243674.png)

#### KeyValue格式

![image-20210319165903842](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210319165903842.png)

- 开始的`key length` 和 `value length` 为键的长度和值的长度，因此可以直接访问value部分；
- 可以发现 key 部分存储了指定单元的全维度内容，包含了 行键、列族名和列限定符等，因此如果值较小时，应当也保持键尽量小，保证键值的比率合适；

### WAL

![image-20210322111241181](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210322111241181.png)

#### HLog类

- 实现了WAL的类是 `HLog` 。

- 为了提高性能，在`put`、`delete` 、`increment`时可以调用`setWriteToWAL(false)` 来停止写入日志，这样可以获得额外的性能，但也就失去了恢复数据的功能。

- 同一个服务器中的所有`Region`共享同一个`HLog`实例，数据会按照到达的顺序写入到WAL中。

  ![image-20210322112556331](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210322112556331.png)

#### HLogKey类

- WAL使用的是Hadoop的`SequenceFile`， 这种文件格式按照 键/值 集合的方式存储记录；
- `HLogKey` 中存储了`KeyValue`的归属：`Region`和表明，还存储了WAL日志的序列号；
- `HLogKey` 中还存储了记录写入时间；

#### WALEdit类

为了保证每行数据中多个单元格修改时的原子性，可以把包含多个单元格的更新写到一个`WALEdit`实例中。

#### LogSyncer类

- 表的描述符允许用户设置 `延迟日志刷写`；
- 该值默认为 false，意味着每一个的编辑都会调用写日志的 `sync()` 方法，该调用会等待写入日志的确定；
- 可以选择稍微延迟这个`sync()`调用，并让它在后台执行，但在服务器发生故障时，可能会丢失数据；
- 该操作只作用于 用户表；

#### LogRoller类

`LogRoller`类会作为一个后台线程运行，并且在特定的时间间隔内滚动日志。

### 读路径



### Region查找

为了查询特定主键的region，在HBase中提供了2张特殊的目录表 `-ROOT-` 和 `.META.`【在新版本中已经没有`-ROOT-`】，`-ROOT-`用来查询所有 `.META.` 表中 region 的位置，`-ROOT-`表的region从不进行拆分，从而保证类似于 B+树结构的三层查找结构：

- 第一层是`ZooKeeper`中包含root region位置信息的节点；
- 第二层是从`-ROOT-`表中查找对应 meta region的位置;
- 第三层是从`.META.`表中查找用户表对应region的位置。

> 当 `.META.` 的Region大小为128MB时，它可以定位2^34^个Region，或2^61^字节的数据，在数据量增加时，还可以调高Region的大小；



当客户端的缓存为空时，需要3次额外的网络请求 + 1次的get请求：

![image-20210323100152101](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210323100152101.png)

而如果缓存不为空，但缓存的 `User Table Region`、`.META Region` `-ROOT- Region` 的位置都失效时，则需要3次失败的查询，共需要6次网络请求 + 1次的get请求：

![image-20210323100919329](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210323100919329.png)

### Region生命周期

![image-20210323101218967](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210323101218967.png)

Region 状态的改变可能由master发起，也可能由Region Server 发起。例如，当master把 region 分配到一个服务器后，由服务器完成打开过程；拆分过程有 Region Server 发起，该过程会引起一系列的 region 关闭和打开事件。

由于事件都是分布式的，服务器使用 `ZooKeeper` 来跟踪一个特定 znode 的状态。

## 3、高级用法

### 行键设计

在HBase中，筛选的效率从左到右明显下降，所有在KeyValue设计时可以考虑把一些重要的筛选信息左移到合适的位置，从而提高数据的查询性能。

![image-20210323104145876](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210323104145876.png)

### 版本管理

## 4、性能优化

### 垃圾回收优化

用户可以调整Region服务器的垃圾回收参数，由于master没有处理任何过重的负载并且实际的数据服务不会经过它，因此只需要调整 Region 服务器的参数。

当写入负载过大时，memstore在不同时期创建并释放着各种不同大小的对象，这些对象很根据在内存中停留时间的长短被保存在Java 堆中的不同位置：

- 新插入的数据会被分配到 年轻代，该空间可以迅速被回收，对内存管理没有影响；
- 如果数据在内存中停留时间过长，数据会被提升到老年代。

年轻代与老年代的不同点在于：年轻代占用的空间在 128MB到512MB之间，而老年代的空间通常是好几GB。

> 指定新生代空间可以通过以下两种方式完成：
>
> - `-XX:MaxNewSize=128m -XX:NewSize=128m`
> - `-Xmn128m`

> 默认的新生代空间对于大多数region服务器来说都太小，所以它必须增大。否则，可能会发现服务器CPU的使用了会急剧上升，因为新生代的GC会消耗大量的CPU。

GC的停顿时间不能过长：

- 一方面会影响服务器的响应延迟；

- 另一方面如果停顿时间达到了 ZooKeeper 会话的超时限制，服务器会被 master 认为已经崩溃并随后会被抛弃。

### 客户端的最佳实践

- **禁止自动刷写**，当有大量的写入操作时，使用`setAutoFlush(false)` 方法来关闭HTable的自动刷写。
- **使用扫描缓存**，如果HBase被用作MapReduce作业的输入源，可以调用`setCaching()`方法设置比默认值1大的值，这样服务端可以一次性传输多行数据到客户端进行处理。但也要考虑客户端和服务端的内存开销。
- **限定扫描范围**，当使用Scan扫描数据时，可以限制列族和列族中特定的列来减少返回的数据量。
- **关闭`ResultScanner`**，如果没有关闭由`HTable.getScanner()` 返回的 `ResultScanner` 实例，会造成服务端的性能问题。
- **使用块缓存**，能够通过`setCacheBlocks()` 方法来设置使用 Region 服务器中的块缓存，对于那些频繁访问的行，应该使用块缓存。
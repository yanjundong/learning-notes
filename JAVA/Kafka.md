# 一、概述

## 定义

Kafka是一个**分布式**的基于**发布/订阅模式**的**消息队列**，主要应用于**大数据实时处理领域**。

## 消息队列

### 使用消息队列的好处

- 解耦
- 削峰
- 异步通信

### 消息队列两种模式

1. 点对点模式（一对一，消费者主动拉取数据，消息收到后消息清除）

   ![image-20210325101301911](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210325101301911.png)

2. 发布/订阅模式（一对多，消费者消费数据之后不会清除消息）

   ![image-20210325101319576](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210325101319576.png)


## 安装

- 安装Java1.8

- 安装ZooKeeper

  ```sh
  #!/bin/bash
  # ZK群起脚本
  case $1 in
  "start"){
  	for item in xk1301the001 xk1301the002 xk1301the003
  	do
  		echo "************$item start************"
  		ssh $item "/usr/module/apache-zookeeper-3.5.8-bin/bin/zkServer.sh start"
  	done
  };;
  
  "stop"){
  	for item in xk1301the001 xk1301the002 xk1301the003
  	do
  		echo "************$item stop************"
  		ssh $item "/usr/module/apache-zookeeper-3.5.8-bin/bin/zkServer.sh stop"
  	done
  };;
  esac
  ```

- 安装Kafka

  [下载kakfa](https://archive.apache.org/dist/kafka/2.0.0/kafka_2.11-2.0.0.tgz)之后，解压，然后修改配置文件`./config/server.properties`：

  ```properties
  # 当前机器在集群中的唯一标识
  broker.id=0
  # 消息存放的目录
  log.dirs=/usr/module/kafka-2.0.0/data
  # zk的配置
  zookeeper.connect=xk1301the001:2181,xk1301the002:2181,xk1301the003:2181
  ```
  
  ```sh
  #!/bin/bash
  # kafka 的群起脚本
  case $1 in
  "start"){
  	for item in xk1301the001 xk1301the002 xk1301the003
  	do
  		echo "************$item start************"
  		ssh $item "/usr/module/kafka-2.0.0/bin/kafka-server-start.sh -daemon /usr/module/kafka-2.0.0/config/server.properties"
  	done
  };;
  
  "stop"){
	for item in xk1301the001 xk1301the002 xk1301the003
  	do
  		echo "************$item stop************"
  		ssh $item "/usr/module/kafka-2.0.0/bin/kafka-server-stop.sh"
  	done
  };;
  esac
  
  ```
  
  

# 二、架构

## 架构

![image-20210326172016136](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210326172016136.png)



- `Producter`：消息生产者，向 kafka broker 发消息的客户端；
- `Consumer`：消息消费者，从kafka broker 取消息的客户端；
- `broker`：一台 kafka 服务器就是一个 broker。一个集群由多个 broker 组成。一个 broker
  可以容纳多个 topic。
- `topic`：kafka中的消息都是以主题为单位进行归类，生产者和消费者面向的都是一个 topic；
- `Partition`：一个topic可以细分为多个分区，一个分区只会属于单个主题。同一主题下的不同分区所包含的消息是不同的，kafka只保证分区有序而不保证主题有序；
- `Leader`：每个分区多个副本的“主”，生产者和消费者只与leader进行交互；
- `Follower`：每个分区多个副本中的“从”  ，只负责消息的同步，很多时候follower副本中的消息相对leader而言会有一定的滞后；

## 数据的可靠性保证

kafka引入了分区的多副本机制来提高容灾能力，采用了`一主多从` 的关系。在工作时，生产者和消费者只与leader进行交互，而follower只负责消息的同步，很多时候follower中的消息相对leader而言会有一定的滞后。

所有与leader保持一定程度同步的flower会组成 `ISR(In-Sync Replicas)`，leader会维护和跟踪ISR中所有follwer的滞后状态，当follower副本落后太多或失效时，leader副本会把它从ISR集合中剔除。

![image-20210329153728411](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210329153728411.png)

- `HW(High Watermark)` 标识了一个特定的消息偏移量，消费者只能拉取到这个偏移量之前的消息；
- `LEO(Log End Offser)` 标识了当前日志文件中下一条待写入消息的偏移量，LEO的大小相当于当前分区中最后一条消息的偏移量+1；
- 分区ISR集合中的每个副本都会维护自身的`LEO`，而**ISR集合中最小的LEO即为分区的HW**；
- Kafka 的复制机制既不是完全的同步复制，也不是单纯的异步复制。kafka使用的这种`ISR`的方式则有效地权衡了数据可靠性和性能之间的关系。

# 三、生产者

## 分区策略



## ACK应答机制

Kafka提供了`acks` 参数来指定分区中必须要有多少个副本收到该消息，用户可以根据对可靠性和吞吐量的要求进行权衡，调节该参数：

- `acks=1` ：默认值，producer 等待 broker 的 ack， partition 的 leader 落盘成功后返回 ack，如果在 follower
  同步成功之前 leader 故障，那么将会丢失数据；  
- `acks=0`：producer 不等待 broker 的 ack，这一操作提供了一个最低的延迟， broker 一接收到还
  没有写入磁盘就已经返回，当 broker 故障时有可能**丢失数据**；  
- `acks=-1`：producer 等待 broker 的 ack， partition 的 leader 和 follower 全部落盘成功后才
  返回 ack。但是如果在 follower 同步完成后， broker 发送 ack 之前， leader 发生故障，那么会
  造成**数据重复**。  

# 四、消费者

## 消费者与消费组

- 消费者负责订阅Kafka中的主题，并从订阅的主题上拉取消息，同时每个消费者都会有一个对应的消费组。

- 每条消息只会被投递给消费组中的一个消费者；

  ![image-20210330112901990](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210330112901990.png)

- 消息中间件会有两种消息投递模式：点对点模式和发布/订阅模式，

  - 当所有的消费者都隶属于同一个消费组，每条消息只会被一个消费者处理，这就相当于**点对点模式**；
  - 当所有的消费者都隶属于不同的消费组，每条消息会被所有的消费者处理，这就相当于**发布/订阅模式**；

- 消费组时一个逻辑上的概念，每一个消费者只会隶属于一个消费组。每个消费组都有一个固定的名称。

## 消费消息的offset

在每次调用 `poll()` 方法时，它仅仅返回没有被消费过的消息集，因此就需要记录上一次消费的offset，在新的消费者客户端中，该内容保存在Kafka内部的主题`_consumer_offsets`中，在旧版客户端中保存在`ZooKeeper` 中。

消费者在消费完消息后需要执行offset的提交，方法分为自动提交和手动提交，可以通过`enable.auto.commit` 进行配置，默认值为 true：

- 自动提交：每隔5秒会将拉取到的每个分区中最大的消息偏移量进行提交，该动作是在 `poll()` 方法中完成的；会造成**重复消费**和**丢失数据**问题；
- 手动提交：更加灵活，可以根据业务在合适的地方进行提交；可以细分为**同步提交**和**异步提交**；

## 再均衡

**再均衡**是指分区的所属权从一个消费者转移到另一消费者，它为消费组具备高可用性和伸缩性提供保障。

但是再均衡期间消费组会变成不可用，而且还会有重复消费的可能，因此应尽量避免不必要的 再均衡 的发生。

在`KafkaConsumer#subscribe()` 的参数中可以提供一个再均衡监听器 `ConsumerRebalanceListener` ，它可以再均衡监听器用来设定发生再均衡动作前后的一些准备或收尾的动作，该接口有2个方法：

```java
/*
	该方法会在再均衡开始之前和消费者停止读取消息之后被调用。可以通过这个回调方法来处理消费位移的提交，以此		来避免一些不必要的重复消费现象的发生。参数partitions表示再均衡前所分配到的分区。
*/
void onPartitionsRevoked(Collection<TopicPartition> partitions);
/*
	该方法会在重新分配分区之后和消费者开始读取消费之前被调用。参数partitions表示再均衡后所分配到的分区。
*/
void onPartitionsAssigned(Collection<TopicPartition> partitions);
```



# 五、Kafka API

## Producer API

### 消息发送流程

![image-20210330102003052](https://gitee.com/yanjundong97/picBed/raw/master/images/image-20210330102003052.png)

- `RecordAccumulator` 主要用于缓存消息以便 `Sender线程`可以批量发送，该缓存可以通过`buffer.memory` 配置，默认值为32MB；
- `ProducerBatch` 可以包含一个或多个`ProducerRecord` ，将较小的`ProducerRecord`拼凑成一个较大的`ProducerBatch`，可以减少网络请求的次数以提升整体的吞吐量；
- 在`RecordAccumulator`内部有一个`BufferPool` ，它用来实现 `ByteBuffer` 的复用。不过`BufferPool` 只针对特定大小的 `ByteBuffer` 进行管理，而不会缓存其他大小的 `ByteBuffer` ，这个特定的大小由`batch.size`参数来指定，默认值为16KB；
- `ProducerBatch`的大小和`batch.size`参数也有着密切的关系，在新建`ProducerBatch`时会评估这条消息的大小是否超过`batch.size`参数的大小，如果不超过，那么就以 `batch.size` 参数的大小来创建`ProducerBatch`，这样在使用完这段内存区域之后，可以通过`BufferPool` 的管理来进行复用；如果超过，该区域就不会被复用；
- `Sender线程` 会从 `RecordAccumulator` 中获取消息，然后会进一步封装为 `<Node, List<OroducerBatch>>` ，其中 `Node` 表示 Kafka 集群的 broker 节点。在这之后，`Sender `还会进一步封装成`<Node，Request>`的形式，这样就可以将Request请求发往各个Node了；

### 初始化配置

```java
    private static final String BROKER_LIST = "192.168.1.149:9092";

    public static Properties initConfig() {
        Properties props = new Properties();
        // 指定生产者客户端连接Kafka集群所需的broker地址清单
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, BROKER_LIST);
        // 指定 key 和 value 序列化器的全限定名
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, 
                StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
                StringSerializer.class.getName());
        // KafkaProducer对应的客户端id
        props.put(ProducerConfig.CLIENT_ID_CONFIG, "producer-1");
        return props;
    }
```

### 发送即忘

```java
    public static void main(String[] args) {
        Properties props = initConfig();
        KafkaProducer<String, String> producer = new KafkaProducer<>(props);
        try {
            for (int i = 100; i > 0; i--) {
                producer.send(new ProducerRecord<>("topic-first", "value" + i));
            }
        } finally {
            producer.close();
        }
    }
```

这种方法只管往Kafka中发送消息而并不关心消息是否正确到达。该方法性能最高，但可靠性最差。

### 异步发送消息

```java
    public static void main(String[] args) {
        Properties props = initConfig();
        KafkaProducer<String, String> producer = new KafkaProducer<>(props);
        try {
            for (int i = 100; i > 0; i--) {
                producer.send(new ProducerRecord<>("topic-first", "value" + i), new Callback() {
                    @Override
                    public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                        if (e != null) {
                            e.printStackTrace();
                        } else {
                            System.out.println(recordMetadata);
                        }
                    }
                });
            }
        } finally {
            producer.close();
        }
    }
```

- `KafkaProducer#send(ProducerRecord<K, V> record, Callback callback)` 方法可以指定一个`Callback` 的回调函数，在返回响应时会调用该函数。
- 对于同一个分区而言，如果消息`record1`在`record2`之前先发送，`KafkaProducer`就可以保证对应的`callback1`在`callback2`之前调用。

### 同步发送消息

```java
public static void main(String[] args) {
    Properties props = initConfig();
    KafkaProducer<String, String> producer = new KafkaProducer<>(props);
    try {
        for (int i = 100; i > 0; i--) {
            // .get() 的调用
            producer.send(new ProducerRecord<>("topic-first", "value" + i)).get();
        }
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    } finally {
        producer.close();
    }
}
```

`KafkaProducer#send()` 方法的返回值为`Future<RecordMetadata>` ，利用这个返回值就可以同步发送。

### 分区器

分区器的作用就是为消息分配分区，Kafka中提供的默认分区器是`DefaultPartitioner`，如果该类不能满足需求，还可以自定义分区器，只需要实现`org.apache.kafka.clients.producer.Partitioner` 即可。

## Consumer API

### 初始化配置

```java
private static final String BROKER_LIST = "192.168.1.149:9092";
private static final String TOPIC = "topic-first";
private static final AtomicBoolean isRunning = new AtomicBoolean(true);

public static Properties initConfig() {
    Properties props = new Properties();
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, BROKER_LIST);
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
              StringDeserializer.class.getName());
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
              StringDeserializer.class.getName());
    props.put(ConsumerConfig.GROUP_ID_CONFIG, "group-1");
    props.put(ConsumerConfig.CLIENT_ID_CONFIG, "consumer-1");
    return props;
}
```

### 消费消息

```java
public static void main(String[] args) {
    Properties props = initConfig();
    KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
    consumer.subscribe(Arrays.asList(TOPIC));
    try {
        while (isRunning.get()) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(2));
            for (ConsumerRecord<String, String> record : records) {
                System.out.println("topic = " + record.topic() +
                                   ", partition = " + record.partition() +
                                   ", offset = " + record.offset() +
                                   ", key = " + record.offset() +
                                   ", value = " + record.value());
            }
        }
    } finally {
        consumer.close();
    }
}
```

### 订阅主题

订阅主题的方法如下：

```java
public void subscribe(Collection<String> topics);
public void subscribe(Collection<String> topics, ConsumerRebalanceListener listener);
public void subscribe(Pattern pattern, ConsumerRebalanceListener listener);
/*
	使用正则表达式的方式订阅，如果创建了新的主题，并且主题名称与正则表达式相匹配，那么也会消费该主题的消息
*/
public void subscribe(Pattern pattern);
```

消费者还可以通过`KafkaConsumer#assign(Collection<TopicPartition> partitions)` 的方法订阅主题的某个分区，其中`TopicPartition` 表示分区。

### 指定offset消费

`KafkaConsumer` 中的 `seek()`方法可以指定开始消费消息的偏移量：

```java
/*
	需要指定 partition，该方法也只能重置消费者分配到的分区，所以在执行该方法前需要先执行一次 poll() 方	法，等待分配到分区后才可以重置位置。
*/
public void seek(TopicPartition partition, long offset);
```

```java
Properties props = initConfig();
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList(TOPIC));

Set<TopicPartition> assignment = new HashSet<>();
while (assignment.isEmpty()) {  // 如果不为空，说明已经分配到了分区
    consumer.poll(Duration.ofSeconds(1));
    assignment = consumer.assignment();
}
for (TopicPartition tp : assignment) {
    consumer.seek(tp, 10);
}
while (isRunning.get()) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(2));
    // consume the record
}
```

`KafkaConsumer` 中还提供了 `endOffsets()` 和 `beginningOffsets()` 方法来获取指定分区的起始和末尾消息位置。

用户也可以通过`KafkaConsumer#offsetsForTimes()` 方法实现类似**消费昨天8点之后的消息**这样的功能。

```java
Properties props = initConfig();
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList(TOPIC));

Set<TopicPartition> assignment = new HashSet<>();
while (assignment.isEmpty()) {  // 如果不为空，说明已经分配到了分区
    consumer.poll(Duration.ofSeconds(1));
    assignment = consumer.assignment();
}
Map<TopicPartition, Long> timestampToSearch = new HashMap<>();
LocalDateTime dateTime = LocalDateTime.of(2021, 3, 29, 20, 0);
long timeStamp = dateTime.toEpochSecond(ZoneOffset.of("+8"));
for (TopicPartition tp : assignment) {
    timestampToSearch.put(tp, timeStamp);
}
Map<TopicPartition, OffsetAndTimestamp> offsets = 
    consumer.offsetsForTimes(timestampToSearch);
for (TopicPartition tp : assignment) {
    OffsetAndTimestamp offsetAndTimestamp = offsets.get(tp);
    if (offsetAndTimestamp != null) {
        consumer.seek(tp, offsetAndTimestamp.offset());
    }
}
```



## 序列化与反序列化

### 序列化

Kafka提供了一些常用的序列化工具，如 `StringSerializer`，`IntegerSerializer`、`LongSerializer`等，它们都实现了 `org.apache.kafka.common.serialization.Serializer` 接口：

```java
void configure(Map<String, ?> var1, boolean var2);

byte[] serialize(String var1, T var2);

void close();
```

### 反序列化

Kafka提供了一些常用的反序列化工具，如 `StringDeserializer`，`IntegerDeserializer`、`LongDeserializer`等，它们都实现了 `org.apache.kafka.common.serialization.Deserializer` 接口：

```java
void configure(Map<String, ?> configs, boolean isKey);
T deserialize(String topic, byte[] data);
void close();
```

## 拦截器

### 生产者拦截器

**生产者拦截器**既可以用来在消息发送前做一些准备工作，比如按照某个规则过滤不符合要求的消息、修改消息的内容等，也可以用来在发送回调逻辑前做一些定制化的需求，比如统计类工作。

用户指定多个 `interceptor`，按序作用于同一条消息从而形成一个拦截链(`interceptor chain`)。 `Intercetpor` 的实现接口是
`org.apache.kafka.clients.producer.ProducerInterceptor`，其定义的方法包括：

```java
/*
	KafkaProducer会在消息序列化和计算分区之前调用该方法，一般来说最好不要修改消息 ProducerRecord 的 		topic、key 和partition 等信息，如果要修改，则需确保对其有准确的判断，否则会与预想的效果出现偏差。比	   如修改key不仅会影响分区的计算，同样会影响broker端日志压缩（Log Compaction）的功能。
*/
public ProducerRecord<K, V> onSend(ProducerRecord<K, V> record);
/*
	KafkaProducer 会在消息被应答之前或消息发送失败时调用该方法，会优先于用户设定的 Callback 之前执行。	 这个方法运行在Producer 的 I/O 线程中，所以这个方法中实现的代码逻辑越简单越好，否则会影响消息的发送速	  度。
*/
public void onAcknowledgement(RecordMetadata metadata, Exception exception);
/*
	该方法主要用于在关闭拦截器时执行一些资源的清理工作。
*/
public void close();
```

> 在拦截链中，如果某个拦截器执行失败，那么下一个拦截器会接着从上一个执行成功的拦截器继续执行。

### 消费者拦截器

**消费者拦截器**主要在消费到消息或在提交消费位移时进行一些定制化的操作。

定义消费者拦截器只需要实现 `org.apache.kafka.clients.consumer.ConsumerInterceptor`，其方法包括：

```java
/*
	会在poll()方法返回之前调用该方法来对消息进行相应的定制化操作，比如修改返回的消息内容、按照某种规则过滤消息
*/
public ConsumerRecords<K, V> onConsume(ConsumerRecords<K, V> records);
/*
	会在提交完消费位移之后调用该方法，可以使用这个方法来记录跟踪所提交的位移信息
*/
public void onCommit(Map<TopicPartition, OffsetAndMetadata> offsets);
public void close();
```

# 六、主题与分区

## KafkaAdminClient

一般情况下都是使用 `kafka-topics.sh` 脚本来管理主题，但在某些特殊情况下，则需要使用 `KafkaAdminClient` 类以通过调用API的方法来实现。

`KafkaAdminClient` 可以用来管理 `broker`、配置和 `ACL(Access Control List)`、主题。

```java
// 创建主题
CreateTopicsResult createTopics(final Collection<NewTopic> newTopics,
                                           final CreateTopicsOptions options);
// 删除主题
DeleteTopicsResult deleteTopics(Collection<String> topicNames,
                                           DeleteTopicsOptions options);
// 列出所有可用的主题
ListTopicsResult listTopics(final ListTopicsOptions options);
// 查看主题信息
DescribeTopicsResult describeTopics(final Collection<String> topicNames,
                                    DescribeTopicsOptions options) ;
// 查询配置信息
DescribeConfigsResult describeConfigs(Collection<ConfigResource> configResources, 
                                      final DescribeConfigsOptions options);
// 修改配置信息
AlterConfigsResult alterConfigs(Map<ConfigResource, Config> configs, 
                                final AlterConfigsOptions options);
// 增加分区
CreatePartitionsResult createPartitions(Map<String, NewPartitions> newPartitions,
                                                   final CreatePartitionsOptions options);
```

> 目前Kafka**不支持减少分区**，因为实现该功能需要考虑的因素太多，如删除分区中的消息如何处理，是直接删除，还是直接插入现有分区的尾部；如果直接插入尾部的话，消息的时间戳就不会递增，这样就会影响Spark、Flink这样的组件，因此没有支持该功能。如果真的需要实现此类功能，则完全可以重新创建一个分区数较小的主题，然后将现有主题中的消息按照既定的逻辑复制过去即可。

## 分区的管理

### 优先副本的选举

为了解决负载失衡的问题，Kafka引入了 `优先副本` 的概念，优先副本是指 AR 集合列表中的第一个副本，Kafka要确保所有主题的优先副本在集群中均匀分布，这就保证了所有分区的leader均衡分布。

默认情况下，Kafka会自动分区平衡，可以通过参数 `auto.leader.rebalance.enable` 配置。如果开启分区自动平衡的功能，则 Kafka 的控制器会启动一个定时任务，这个定时任务会轮询所有的 broker节点，计算每个broker节点的分区不平衡率（broker中的不平衡率=非优先副本的leader个数/分区总数）是否超过`leader.imbalance.per.broker.percentage`参数配置的比值，默认值为 10%，如果超过设定的比值则会自动执行优先副本的选举动作以求分区平衡。

在生产环境下，不建议将`auto.leader.rebalance.enable`设置为默认的true，因为分区均衡执行的时间不能自主掌握。

### 分区重分配

当要对集群中的一个节点进行有计划的下线操作时，为了保证分区及副本的合理分配，我们希望通过某种方式能够将该节点上的分区副本迁移到其他的可用节点上。

当集群中新增了 broker 节点时，新节点的负载和原先节点的负载之间会严重不均衡。

为了解决上面的问题，需要对分区副本重新进行合理的分配，该功能可以通过`kafka-reassign-partitions.sh` 脚本来实现。

### 修改副本因子

创建主题之后，仍然可以修改副本因子，该功能可以通过 `kafka-reassign-partition.sh` 脚本来实现。



《深入理解Kafka：核心设计与实践原理》 第5章 日志存储
















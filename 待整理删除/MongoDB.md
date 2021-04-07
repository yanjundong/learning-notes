# MongoDB

## BASE操作

传统的关系型数据库中，**事务** 是个很重要的功能，它能够保证所有的记录数据要么被处理，要么所有的记录数据不被处理，保证了数据库的ACID特征。

​	ACID特征

* 原子性
* 一致性
* 隔离性
* 永久性

但 MongoDB在设计时，考虑到事务带来的低效的问题，就没有加入事务。在MongoDB中加入了单文档原子性、多文档原子性操作功能，部分解决了数据丢失、出错的问题。

### 单文档原子性操作

是指对集合中的单个文档进行操作时，确保插入、更新和删除等操作要么成功，要么不成功，避免出现只操作了一半的情况。

```sql
use eshops
db.shoppingcart.insert(
    {_id:100100001,
    goodsName:"《python语言》",
    price:40,
    amoutn:1,
    init:"元",
    recordtime:ISODate("2017-09-24"),
    flag:false,
    checkout:[{by:"user",enddate:ISODate("2017-09-24")}]}
)


db.shoppingcart.findAndModify(
    {
    query:{_id:100100001},
    update:{
    	"$set":{flag:1},
    	"$push":{checkout:{by:"张三",enddate:new Date()}}
    }
    }
)
```



### 多文档隔离性操作



在update等操作时，在指定查询条件的字段上设置$isolated:1，满足条件的文档在执行update操作时就具有隔离性。也就是说在执行此命令期间，其他用户的进程不能够对这些文档进行读写操作。

> $isolate 的局限性
> 1. 当更新过程出错时，不执行回滚机制；
> 2. 不支持分片操作

## 高级索引和索引限制

建立索引的唯一目的就是提高查询效率。

### 高级索引

高级索引只要实现对文档中的子文档和数组建立索引，实现**地理空间数据索引**

#### 子文档索引

```sql
db.users.insertMany([
    {
   "address": {
      "city": "yuncheng",
      "state": "shanxi",
      "pincode": "456"
   },
   "tags": [
      "football",
      "cricket",
      "blogs"
   ],
   "name": "yanjundong"
},{
   "address": {
      "city": "Los Angeles",
      "state": "California",
      "pincode": "123"
   },
   "tags": [
      "music",
      "cricket",
      "blogs"
   ],
   "name": "Tom Benzamin"
}
])
```

```sql
//建立索引
db.users.createIndex({
                     "address.city":1,"address.state":1,"address.pincode":-1
                     })
```

```sql
//建立索引之后，我们可以使用子文档的字段来检索数据
db.users.find({"address.state":"California","address.city":"Los Angeles"}) 
```

#### 数据索引

```sql
db.users.createIndex({"tags":1})
```

```sql
db.users.find({tags:"football"})
```

#### 2dsphere索引（地理空间索引）

支持所有MongoDB数据库地理空间查询：包含、交集和领近的查询；并支持GeoJSON存储格式的数据。

```sql
//具体使用参考 使用白皮书
```

### 索引限制

1. 建立索引会有额外的开销，一个索引至少需要占用8KB的空间，即需要消耗内存和磁盘的存储空间。另外，在对集合进行插入、更新和删除时，如果相关字段建立了索引，同步也会引起索引的更新操作（锁独占排他性操作），影响数据库的读写性能。

2. 索引在使用的时候，是驻内存持续运行的。因此需要考虑到内存的限制。

3. 索引不能被以下的查询使用：

   a. 正则表达式及非操作符，如`$nin`、`$not`等；

   b. 算术运算符，如`$mod`等；

   c. `$where`子句；

4. 索引的最大范围，集合中索引不能超过64个，索引名的长度不能超过125个字符，一个多值索引最多可以有31个字段。

5. 不应该使用索引的场景

   a. 如果查询返回的结果超过集合文档的1/3，那么是否建立索引，需要慎重考虑；

   b. 对于以写为主的集合，慎用索引，_id就够用了；

## 查询高级分析

为了测试索引的性能，`find()` 提供了 `Explan()`、`Hint()`等方法来检查和测试

### Explan()分析

通过对`find()`、`aggregate()`、`count()`、`distinct()`、`group()`、`remove()`、`update()`命令执行结果的分析，判断索引是否可靠。

语法：

```sql
db.Collection.Command.explain(modes)
db为当前数据库
Collection为需要操作的集合名
Commond为上述find()、update()等命令
modes参数为"queryPlanner"、"executionStats"、"allPlansExecution"
1. queryPlanner 模式
2. executionStats 模式
3. allPlansExecution 模式
```

```sql
//执行命令
db.testCollecion.find().explain("executionStats")
//运行结果
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "eshops.testCollecion",
                "indexFilterSet" : false,
                "parsedQuery" : {

                },
                "winningPlan" : {
                        "stage" : "EOF"
                },
                "rejectedPlans" : [ ]
        },
        "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 0,
                "executionTimeMillis" : 16,
                "totalKeysExamined" : 0, //扫描的索引条目数量；扫描索引条目的数量越少越好
                "totalDocsExamined" : 0, //扫描文档的数量；扫描文档的数量越少越好
                "executionStages" : {
                        "stage" : "EOF",
                        "nReturned" : 0,
                        "executionTimeMillisEstimate" : 0,
                        "works" : 1,
                        "advanced" : 0,
                        "needTime" : 0,
                        "needYield" : 0,
                        "saveState" : 0,
                        "restoreState" : 0,
                        "isEOF" : 1,
                        "invalidates" : 0
                }
        },
        "serverInfo" : {
                "host" : "1301-zhuo-PC",
                "port" : 27017,
                "version" : "3.4.4",
                "gitVersion" : "888390515874a9debd1b6c5d36559ca86b44babd"
        },
        "ok" : 1
}
```

### Hint()分析

可以使用`hint`来强制MongoDB使用一个指定的索引。

`hint`可以为查询临时指定需要索引的字段，用法如下

1. 强制指定一个索引Key，如：db.collection.find().hint("age_1");

2. 强制对集合做正向扫描或反向扫描，如

   db.users.find().hint({$natural:1})  //执行正向扫描

   db.users.find().hint({$natural:-1})  //执行反向扫描

```sql
//执行命令
db.users.find({"address.state":"California"}).hint({_id:1}).explain("executionStats")//基于_id正向扫描
//运行结果
 "executionTimeMillis" : 22,
 "totalKeysExamined" : 2,
 "totalDocsExamined" : 2,
 
//执行命令
db.users.find({"address.state":"California"}).explain("executionStats")
//运行结果
 "executionTimeMillis" : 0,
 "totalKeysExamined" : 0,
 "totalDocsExamined" : 2,
 
 
//显然，在大数据环境下，后者的执行效率更高
```

## 案例实战

### 订单信息记录

 MongoDB记录订单信息的优势是：

1. 能够处理海量的数据；
2. 读写操作的响应性能好

但是对于订单这样高价值的数据，因为NoSQL不对保存的数据进行验证，所以存在风险。

要保证交易数据的ACID特征，必须要考虑具有**事务处理功能**的关系型数据库。为了兼顾高效的响应和数据处理的可靠，可以将两者搭配使用。

> ACID特征
>
> 1. 原子性
> 2. 一致性
> 3. 隔离性
> 4. 持久性

### 原子性修改

`findOneAndUpdate`

```java
@Test
	public void queryUserTest() {
		Document doc = mongoClient
				.getDatabase("UserTest").getCollection("user")
				.findOneAndUpdate(new Document("userName", "zhangsan"),new Document("$set",new Document("age","24").append("sex","女")));
		if (doc != null)
			System.out.println(doc.toJson());
		else
			System.out.println("查询结果为空！");
	}
```

```java
//输出结果
{ "_id" : { "$oid" : "5db05f118233fe1790881c05" }, "userName" : "zhangsan", "nickName" : "小明", "name" : "张三", "age" : "21", "sex" : "男", "school" : "清华大学", "address" : "中国北京", "codeNum" : "123456789012345678", "phone" : "13000012121" }
```

### 定期批量转移

对于大量的订单数据，应该定期的转移处理，以保证用户对订单的查询响应速度。

把一个集合中的数据转移到另外一个集合中，MongoDB至少有两种方式：

1. 通过代码编程实现数据的转移；
2. 通过集合数据的导入/导出工具实现批量转移；

```java
////Filters的用法——为什么Filters.text(String search)运行出错？？
public class MongoDBJDBC{
   public static void main( String args[] ){
      try{  
         // 连接到 mongodb 服务
         MongoClient mongoClient = new MongoClient( "localhost" , 27017 );
          
         // 连接到数据库
         MongoDatabase mongoDatabase = mongoClient.getDatabase("xttblog"); 
         System.out.println("Connect to database successfully");
          
         MongoCollection<Document> collection = mongoDatabase.getCollection("test");
         System.out.println("集合 test 选择成功");
          
         //将文档中likes=100的文档修改为likes=200  
         collection.updateMany(Filters.eq("likes", 100), new Document("$set",new Document("likes",200))); 
 
         //将文档中likes=100 or likes=200 的文档修改为likes=300
         collection.updateMany(Filters.or(Filters.eq("likes", 100), Filters.eq("likes", 300)), new Document("$set",new Document("likes",300))); 
 
         //将文档中likes=300 AND _id=200 的文档修改为likes=400
         collection.updateMany(Filters.and(Filters.eq("likes", 100), Filters.eq("id", 200)), new Document("$set",new Document("likes",400)));
 
         //将文档中likes != 100 的文档修改为likes=500
         collection.updateMany(Filters.ne("likes", 100)), new Document("$set",new Document("likes",500)));
 
         //将文档中likes > 100 的文档修改为likes=600
         collection.updateMany(Filters.gt("likes", 100)), new Document("$set",new Document("likes",600)));
 
         //将文档中likes < 800 的文档修改为likes=700
         collection.updateMany(Filters.lt("likes", 100)), new Document("$set",new Document("likes",700)));
         // ......
         //检索查看结果 
         FindIterable<Document> findIterable = collection.find(); 
         MongoCursor<Document> mongoCursor = findIterable.iterator(); 
         while(mongoCursor.hasNext()){ 
            System.out.println(mongoCursor.next()); 
         } 
       
      }catch(Exception e){
         System.err.println( e.getClass().getName() + ": " + e.getMessage() );
      }
   }
}
```

### 统计点击量

```java
    @Test
    public void queryHistoryOrderTest() {
        long count = mongoClient
                .getDatabase("ClicksLogTest").getCollection("clicksLog")
                .count(new Document("clickPosition","p_001"));
        System.out.println(count);
    }
```

### 特点

1. 采用java类封装功能，因为MongoDB的特征（对数值和字符串是不敏感的），需要对类属性的数据类型做严格的约束。例如，价格属性不能输入字符串内容。
2. 需要认识到MongoDB的优点和缺点。对于处理海量的数据，具有响应速度快的优点，同时能够实现对数据的备份查询。但是对于像具有高价值的订单记录等数据处理，需要选择关系型数据库，确保不会出现任何的差错。












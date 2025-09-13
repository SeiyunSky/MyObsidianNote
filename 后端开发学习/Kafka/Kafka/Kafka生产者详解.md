
  ![[Pasted image 20250912102658.png|500]]
  核心是里面的ProducerRecord对象，作为Kafka的核心类，包含了Topic和Partiton两个属性，（Partition和RocketMQ里的Queue非常相似）也需要将键值对对象序列化为字节数组在网络上进行传输。
  根据Topic的配置，决定消息的时间戳是`CREATETIME`还是`LogAppendTime(broker重写)`
  消息抵达后，如果未自定分区，基于KEY的hash函数映射指定分区，循环方式选择分区号的。每个消费者对一个分区进行处理。
   `如果发送过程中指定了有效的分区号，那么在发送记录时将使用该分区。如果发送过程中未指定分区，则将使用key 的 hash 函数映射指定一个分区。如果发送的过程中既没有分区号也没有，则将以循环的方式分配一个分区。选好分区后，生产者就知道向哪个主题和分区发送数据了`
   Broker收到消息后的响应包含了`RecordMetaData对象`，里面有**主题、分区信息、记录在分区里的偏移量和时间戳**
   
RocketMQ在日志模型上的特性区别是：
Kafka 将每个分区视为一个​**​只能追加（Append-Only）的日志文件​**​
```bash
# 磁盘上的物理存储：  
topic-order-0/    ​到达顺序物理存储，​极高的吞吐量​
    ├── 00000000000000000000.log  # 第一个日志段
    ├── 00000000000000000000.index # 索引文件
    ├── 00000000000000123456.log  # 下一个日志段
    └── ...
```
RocketMQ则是队列+索引模型
```bash
# 磁盘上的物理存储：
store/
    ├── commitlog/   commitlog存储所有内容，Queue存储索引，消费延迟低
    │   └── 00000000000000000000  # 所有消息都写在这里
    └── consumequeue/
        ├── topic_order/queue_0/00000000000000000000 # 队列0的索引
        ├── topic_order/queue_1/00000000000000000000 # 队列1的索引
        └── ...
```

   此处的控制一共有三个不同级别的路由控制粒度
   - 指定分区号
   - 指定Key
   - 轮询投递

### **Kafka生产者配置**
   bootstrap.servers  指定broker的地址清单
   key.serializer  value.serializer实现接口进行序列化
**创建生产者**
```java
   Properties properties = new Properties(); 
   
   properties.put("bootstrap.servers", "broker1:9092,broker2:9092"); 
   properties.put("key.serializer", 
   "org.apache.kafka.common.serialization.StringSerializer"); 
   properties.put("value.serializer", 
   "org.apache.kafka.common.serialization.StringSerializer"); 

   KafkaProducer<String, String> producer = new KafkaProducer<>(properties);
   ```

**发送简单消息**
```java
构造函数使用public ProducerRecord(String topic, K key, V value) {}
ProducerRecord<String,String> record = new ProducerRecord<String, String>("CustomerCountry","West","France"); 
producer.send(record);
```
！发送成功后send()会返回一个Future\<RecordMetadata>对象！
**同步发送消息**
```java
ProducerRecord<String,String> record = new ProducerRecord<String, String>("CustomerCountry","West","France"); 
try{ 
    RecordMetadata recordMetadata = producer.send(record).get();
}catch(Exception e){
    e.printStackTrace()；
}
```
在send后调用get方法等待kafka响应，响应成功则会获得对象查看消息记录，send本身是异步方法，但是Future.get（）方法会阻塞当前线程，直到消息发送完成。
![[7c1d60429580e8.png|500]]
发送中可能会出现的问题包括无主和信息量过大
**异步发送数据**
```java
ProducerRecord<String, String> producerRecord = new ProducerRecord<String, String>("CustomerCountry", "Huston", "America"); 

producer.send(producerRecord,new DemoProducerCallBack()); 

class DemoProducerCallBack implements Callback { 
public void onCompletion(RecordMetadata metadata, Exception exception){ 
    if(exception != null){ 
        exception.printStackTrace();; 
    }
}
}
```

**生产者分区策略**
自带顺序轮询-随机轮询
```java
List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
return ThreadLocalRandom.current().nextInt(partitions.size());
```
按照key-ordering策略保存
```java
List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
return Math.abs(key.hashCode()) % partitions.size();
```
自定义配置
```java
public interface Partitioner extends Configurable, Closeable {
 public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster); 
 
 public void close(); default public void onNewBatch(String topic, Cluster cluster, int prevPartition) {
 }
}
```

**生产者压缩机制**
Kafka Producer 中使用 `compression.type` 来开启压缩
```java
private Properties properties = new Properties();
properties.put("bootstrap.servers","192.168.1.9:9092");
properties.put("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
properties.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
properties.put("compression.type", "gzip"); 
Producer<String,String> producer = new KafkaProducer<String, String>(properties); 
ProducerRecord<String,String> record = new ProducerRecord<String, String>("CustomerCountry","Precision Products","France");
```


   


   


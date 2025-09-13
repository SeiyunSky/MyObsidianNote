![[Pasted image 20250912131101.png]]
![[Pasted image 20250912131946.png]]
在相同消费者组下，分区只能由其绑定的消费者进行消费，消费者可以处理多个分区，也可以多消费者群组去处理同一主题下的数据。

消费者如果停止发送心跳了，Broker就会将过期会话的消费者判定为已死亡，而如果消费者宕机且停止发送消息，则会等待若干时间。
在这个过程中，会触发STW，且时间非常长。

**创建消费者**
```java
Properties properties = new Properties(); properties.put("bootstrap.server","192.168.1.9:9092"); properties.put("key.serializer","org.apache.kafka.common.serialization.StringSerializer"); properties.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
KafkaConsumer<String,String> consumer = new KafkaConsumer<>(properties);
```
**主题订阅**
```java
利用不可变集合订阅单一主题
consumer.subscribe(Collections.singletonList("customerTopic"));
订阅全部相关主题
consumer.subscribe("test.*");
```
**轮询**
```java
try { 
   while (true) { 
      ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(100)); 
      for (ConsumerRecord<String, String> record : records) { 
         int updateCount = 1; 
         if (map.containsKey(record.value())) {
            updateCount = (int) map.get(record.value() + 1); 
        } map.put(record.value(), updateCount); 
       } 
    }
}finally { 
   consumer.close();
}
```
- Kafka 必须定期循环请求数据，否则就会认为该 Consumer 已经挂了，会触发重平衡，它的分区会移交给群组中的其它消费者。传给 `poll()` 方法的是一个超市时间，用 `java.time.Duration` 类来表示，如果该参数被设置为 0 ，poll() 方法会立刻返回，否则就会在指定的毫秒数内一直等待 broker 返回数据。
- poll() 方法会返回一个记录列表。每条记录都包含了记录所属主题的信息，记录所在分区的信息、记录在分区中的偏移量，以及记录的键值对。我们一般会遍历这个列表，逐条处理每条记录。

 **特殊偏移**
 我们上面提到，消费者在每次调用`poll()` 方法进行定时轮询的时候，会返回由生产者写入 Kafka 但是还没有被消费者消费的记录，因此我们可以追踪到哪些记录是被群组里的哪个消费者读取的。消费者可以使用 Kafka 来追踪消息在分区中的位置（偏移量）
 **消费者会向一个叫做 `_consumer_offset` 的特殊主题中发送消息**，这个主题会保存每次所发送消息中的分区偏移量，这个主题的主要作用就是消费者触发重平衡后记录偏移使用的，消费者每次向这个主题发送消息，**正常情况下不触发重平衡，这个主题是不起作用的**，当触发重平衡后，消费者停止工作，每个消费者可能会分到对应的分区，这个主题就是让消费者能够继续处理消息所设置的。
 如果提交的偏移量小于客户端最后一次处理的偏移量，那么位于两个偏移量之间的消息就会被重复处理，而大于则会导致数据丢失。
 ![[Pasted image 20250912134644.png]]
 可以通过
 `enable.auto.commit`设置自动偏移量提交
 `enable.commit.interval.ms` 控制提交的时间间隔
 如果不自动提交，可以用`commitAsync()`和`commitSync()`进行提交，前者的异步提交不会进行重试操作，而同步提交会进行一致性重试。

 
 
 
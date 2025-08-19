​**​SQL 过滤和 TAG 过滤是互斥的​​
不能在同一订阅中同时使用这两种过滤方式**
可以通过Tag转换为SQL属性或在SQL内直接使用Tags系统属性来复用

**通过控制subscribe决定消费对象**
```java
//通过选择订阅对应的tag来决定，*为全体tag
//主题——tag
consumer.subscribe("topic1","此处是tag || tag");
```

**sql过滤——只允许消费者接收符合特定条件的消息**
- 生产端
```java
Message msg = new Message(
    "TopicTest", 
    "TagA", 
    "Hello RocketMQ".getBytes());

// 设置用户属性
msg.putUserProperty("a", String.valueOf(1));
msg.putUserProperty("b", "hello");
producer.send(msg);
```
- 消费端
```java
consumer.subscribe("TopicTest", 
    MessageSelector.bySql("a between 0 and 3 AND b = 'hello'"));
```
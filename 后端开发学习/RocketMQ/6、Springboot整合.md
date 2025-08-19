```yml
rocketmq:
    name-server: localhost:9876
    producer:
        group: group1
```
**生产者**
```java
@RequiredConstructor 构造器注入

@Autowired  自动注入
RocketMQTemplate rocketMQTemplate;

//消息转换为字节数组
rocketMQRTemplate.convertAndSend("topic",【想传输的数据】);
```
**消费者**
```java
//通过注解实现监听配置
@RocketMQMessagerListener(topic = "", selectorExpression = "",group = "")
public class Consumer implements RocketMQListener<数据泛型>{

    @Override
    public void onMessage(T t){
        完成任务的代码;   
    }
}
```

其他方法
```java
1 同步发送
SendResult send(String destination, Message<?> message);

2 异步发送
void send(String destination, Message<?> message, SendCallback sendCallback);

3 带tag发送
SendResult send(String destination, String tag, Message<?> message);

4 延迟级别发送
SendResult send(String destination, Message<?> message, long delayMillis);

5 带消息队列选择器的发送
SendResult send(String destination, Message<?> message, MessageQueueSelector selector, Object arg);

6 带属性的发送
SendResult send(String destination, Message<?> message, Map<String, String> properties);

7 单向消息
rocketMQTemplate.sendOneWay(destination,msg)

```


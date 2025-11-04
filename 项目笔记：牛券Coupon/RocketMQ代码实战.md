**添加Maven依赖**
```xml
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>2.3.0</version> 
</dependency>
```

**application.yaml添加配置**
```yaml
rocketmq:
  name-server: #nameServer地址
  producer:
      group: oneCoupon_merchant-admin${unique-name:}-service_common-message-execute_pg
      send-message-timeout: 2000 # 发送超时时间
      retry-times-when-send-failed: 1 # 同步发送重试次数
      retry-times-when-send-async-failed: 1 # 异步发送重试次数
```


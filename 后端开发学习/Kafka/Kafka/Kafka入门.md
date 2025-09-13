**Kafka的消息存放在文件系统上，顺序写磁盘的效率大于随机写内存**
![[Pasted image 20250912095340.png]]**消息**：Kafka 中的数据单元被称为`消息`，也被称为记录，可以把它看作数据库表中某一行的记录。
**批次**：为了提高效率， 消息会`分批次`写入 Kafka，批次就代指的是一组消息。
**主题**：消息的种类称为 `主题`（Topic）,可以说一个主题代表了一类消息。相当于是对消息进行分类。主题就像是数据库中的表。
**分区**：主题可以被分为若干个分区（partition），同一个主题中的分区可以不在一个机器上，有可能会部署在多个机器上，由此来实现 kafka 的`伸缩性`，单一主题中的分区有序，但是无法保证主题中所有的分区有序

![[Pasted image 20250912095914.png]]
在Kafka中，集群哦那个给Zookeeper进行数据持久化管理。生产者push模型将消息发送到Broker，而消费者通过**Poll模式订阅消费信息**，其中使用的优化方式有
 - **零拷贝技术**——基于Linux的sendfile，实现对内核缓冲区与用户缓冲区的跳过，将数据直接输入Socket，减少开销
 - **长轮询**——并非忙等待，而是无消息情况下，等待长时间后再返回，防止频繁返回空数据导致性能丢失
 - **批量压缩**——生产者拉取的消息可以进行批量压缩
 - **批处理数据**——分批进行数据发送，进行顺序读写

#### **核心 API**
Kafka 有四个核心API，它们分别是
- **Producer API**，它允许应用程序向一个或多个 topics 上发送消息记录
- **Consumer API**，允许应用程序订阅一个或多个 topics 并处理为其生成的记录流
- **Streams API**，它允许应用程序作为流处理器，从一个或多个主题中消费输入流并为其生成输出流，有效的将输入流转换为输出流。
- **Connector API**，它允许构建和运行将 Kafka 主题连接到现有应用程序或数据系统的可用生产者和消费者。例如，关系数据库的连接器可能会捕获对表的所有更改

#### Kafka重要配置
- **broker端配置**
  broker.id  每个broker都有默认标识0，在集群中需要是唯一值
  port  默认9092
  zookeeper.connect 
  log.dirs  日志目录，日志存放的磁盘地址
  num.recovery.threads.per.data.dir  每个日志目录默认分配一个线程，修改可提高并行性
  auto.create.topics.enable 是否允许自行创建Topic

### [Kafka生产者详解](obsidian://open?vault=MyObsidianNote-main&file=%E5%90%8E%E7%AB%AF%E5%BC%80%E5%8F%91%E5%AD%A6%E4%B9%A0%2FKafka%2FKafka%E7%94%9F%E4%BA%A7%E8%80%85%E8%AF%A6%E8%A7%A3)

### [Kafka消费者详解](obsidian://open?vault=MyObsidianNote-main&file=%E5%90%8E%E7%AB%AF%E5%BC%80%E5%8F%91%E5%AD%A6%E4%B9%A0%2FKafka%2FKafka%E6%B6%88%E8%B4%B9%E8%80%85%E8%AF%A6%E8%A7%A3)


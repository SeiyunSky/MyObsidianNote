### System
```java
退出当前虚拟机
System.exit(0);

通常情况下作为时间戳使用
System.currentTimeMillis()

简单拷贝
System.arraycopy(数据源数组，起始索引，目的地数组，起始索引，拷贝个数)
```

### Runtime
Runtime表示当前虚拟机的运行环境
```java

获取当前系统的运行环境对象
Runtime.getRuntime();

Runtime.getRuntime().availableProcessors() 获得CPU总数

Runtime.getRuntime().maxMemory() 系统能获得的总内存大小（byte）

Runtime.getRuntime().totalMemory() 系统已经获得的总内存大小（byte）

Runtime.getRuntime().freeMemory() JVM剩余的总内存大小（byte）
```

### Object && Objects
```JAVA

1、toString 返回对象的字符串属性
Object obj = new Object();

String str = obj.toString();

```

### **Record**
一种特殊的类，用于存储数据
- 内容不可变
- 自动生成构造函数和访问方法
- 自动实现equals(),hashcode()和toString();
```java
利用record + 存储容器
可以很方便的存储一些数据集合
比方说：
record Packet(int source, int destination, int timestamp) {}

List<Packet> packets = new ArrayList<>();
packets.add(new Packet(1, 2, 100));
packets.add(new Packet(3, 4, 200));

// 查询所有发往目的地2的数据包
List<Packet> filtered = packets.stream()
                              .filter(p -> p.destination() == 2)
                              .toList();
```
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
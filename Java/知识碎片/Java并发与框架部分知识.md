## 一、反射机制(Reflection)的核心价值与应用
反射是Java的元编程能力，允许在运行时动态加载类、访问私有成员、创建对象及调用方法。其核心价值在于**突破静态编译限制**，实现高度灵活性。典型应用包括：
- **JDBC驱动加载**：`Class.forName("com.mysql.cj.jdbc.Driver")`动态注册驱动
- **框架底层支持**：Spring IoC容器通过反射创建并注入Bean
- **动态方法调用**：如MyBatis的Mapper接口动态实现
- **泛型类型擦除补偿**：获取泛型真实类型（如Gson反序列化）
  
关键代码示例：
```java
// 反射创建对象并调用私有方法
Class<?> clazz = Class.forName("com.example.SecretService");
Constructor<?> constructor = clazz.getDeclaredConstructor();
constructor.setAccessible(true);
Object instance = constructor.newInstance();

Method method = clazz.getDeclaredMethod("hiddenOperation");
method.setAccessible(true);
method.invoke(instance);
```

## 二、动态代理(Dynamic Proxy)的实现与演进

动态代理基于反射实现，分为​**​JDK动态代理​**​和​**​CGLIB字节码增强​**​两类。其核心价值是​**​在不修改源码的情况下增强方法功能​**​，是AOP的基石
### JDK动态代理实现模板
```JAVA
public class LogHandler implements InvocationHandler {
    private final Object target;
    
    public Object createProxy(Object target) {
        return Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            target.getClass().getInterfaces(),
            this
        );
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) {
        System.out.println("[LOG] 调用 " + method.getName());
        return method.invoke(target, args);
    }
}
```
**现代演进​**​：Spring AOP通过注解（如`@Transactional`）声明式实现代理，开发者无需手动创建代理类。
## 三、Spring Bean的本质与线程安全

Bean是由Spring容器管理的Java对象，其核心特性包括：

- ​**​生命周期管理​**​：实例化 → 属性填充 → 初始化 → 使用 → 销毁
- ​**​作用域控制​**​：
    - `singleton`（默认）：单例模式，适用无状态服务
    - `prototype`：每次获取新实例，适用有状态对象
    - `request`/`session`：Web请求/会话级作用域

### 单例Bean的线程安全风险与解决方案
```JAVA
@Service 
public class CounterService {
    private int count = 0;  // 共享状态
    
    public void increment() {
        count++;  // 非原子操作
    }
}
```
![[Pasted image 20250711212641.png]]

1. 优先设计无状态Bean
2. 低竞争场景用原子类（如计数器）
3. 高竞争场景用ReentrantLock（如资源池）
4. 读多写少用StampedLock
5. 分布式系统用Redis/Zookeeper分布式锁


### **volatile：轻量级的可见性解决方案**

volatile 可以看作是一个轻量级的同步机制，它主要解决**可见性**和一定程度的**有序性**问题，但**不保证原子性**。

#### **volatile 的两大作用：**

1. **保证可见性 (Guarantees Visibility)**
    
    - 当一个线程**写入**一个 volatile 变量时，JMM 会强制将该线程工作内存中的值**立即刷新到主内存**。
        
    - 当一个线程**读取**一个 volatile 变量时，JMM 会强制让该线程从**主内存中重新读取**该变量的值，而不是使用其工作内存中的缓存副本。
        
    - **比喻**：volatile 变量就像一个贴在公共布告栏上的公告。任何人写公告（写操作）都必须直接写在布告栏上，任何人读公告（读操作）也必须直接去看布告栏，而不是看自己昨天抄写在小本子上的旧内容。
        
2. **禁止指令重排序 (Prevents Instruction Reordering)**
    
    - volatile 关键字会插入一个“内存屏障”（Memory Barrier），这会阻止编译器和处理器对 volatile 变量相关的读写操作进行重排序，确保代码的执行顺序与程序员编写的顺序在一定程度上保持一致。这在实现一些高级并发模式（如双重检查锁定单例模式）时至关重要。
        

#### **volatile 的局限性：**

- **不保证原子性**：对于复合操作，如 count++，volatile 无能为力。虽然每次读取和写入 count 都是最新的，但在“读取-修改-写入”这三个步骤之间，其他线程仍然可以介入，导致最终结果不正确。
    

#### **使用场景：**

volatile 最适合用于“**一个线程写，多个线程读**”的场景，或者当一个变量的状态变更需要被其他线程立即知晓时。

### **synchronized：重量级的全能解决方案**

synchronized 是一个更强大、更“重量级”的同步机制。它通过“锁（Lock）”或“监视器（Monitor）”来保证在同一时刻，只有一个线程可以执行被它修饰的代码块或方法。

#### **synchronized 的三大作用：**

1. **保证原子性 (Guarantees Atomicity)**
    
    - 这是 synchronized 的核心功能。被 synchronized 包围的代码块（称为“临界区”）会被当作一个不可分割的原子单元来执行。一旦一个线程进入该代码块，它就会持有锁，直到代码块执行完毕并释放锁之前，其他任何线程都无法进入。这完美地解决了 count++ 这样的复合操作问题。
        
2. **保证可见性 (Guarantees Visibility)**
    
    - synchronized 同样能保证可见性。当一个线程**释放锁**时（退出 synchronized 块），JMM 会强制将该线程在临界区内做的**所有**修改都刷新到主内存。
        
    - 当一个线程**获取锁**时（进入 synchronized 块），JMM 会强制让该线程的工作内存失效，使其必须从主内存中重新加载共享变量。
        
3. **保证有序性 (Guarantees Ordering)**
    
    - 由于一个锁在同一时间只能被一个线程持有，这天然地保证了持有同一个锁的多个线程对临界区代码的执行是串行的，从而保证了有序性。
        

#### **使用方式：**

- **修饰实例方法**：锁是当前类的实例对象 (this)。
    
- **修饰静态方法**：锁是当前类的 Class 对象。
    
- **修饰代码块**：锁是 synchronized 关键字后面括号里指定的对象。

## JAVA标准的单例类制作方法

```JAVA
public class SingleTon {
    // 1. volatile 关键字修饰变量 防止指令重排序
    private static volatile SingleTon instance = null;
    // 2. 私有构造函数，防止外部直接 new
    private SingleTon(){}
    // 3. 公共的静态方法，用于获取唯一的实例
    public static SingleTon getInstance(){
        // 4. 第一次检查：如果实例已存在，直接返回，避免不必要的同步
        if(instance == null){
            // 5. 同步代码块，保证线程安全
            synchronized(SingleTon.class){
                // 6. 第二次检查：防止多个线程同时通过第一次检查后重复创建实例
                if(instance == null){
                    // 7. 创建实例
                    instance = new SingleTon();
                }
            }
        }
        return instance;
    }
}
```

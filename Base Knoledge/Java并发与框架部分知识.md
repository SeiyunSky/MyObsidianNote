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


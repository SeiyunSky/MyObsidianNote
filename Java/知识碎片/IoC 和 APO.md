- Spring 框架
 Web: Spring Web MVC、Spring Web Flux
 持久层: Spring Data/JPA、Spring Data Redis、Spring Data MangoDB
 安全校验：SpringSecurity
 构建工程脚手架：Spring Boot
 微服务：SpringCloud

IoC是以全家桶各个功能模块的基础，创建对象的容器
AOP是以IoC为基础，面向切面编程

#### **面向切面编程**
![[Pasted image 20250701170914.png|300]]
将方法中重复性的东西切割下来，抽象化一个新对象来实现原方法中的方法
- 打印日志
- 事务
- 权限处理

## **1.1** **IOC**

  控制反转，将对象的创建进行反转，常规情况下，对象都是开发者手动创建的，使用IoC开发者不再需要创建对象，而是由IoC容器根据需求自动创建项目所需要的对象。
  
  1，pom.xml
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>版本ID</version>
</dependency>

作用：在对象上添加 @Data 的注解，自行生成get，set方法
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>版本ID</version>
</dependency>
```
基于XML和基于注解
  基于XML：
    开发者把需要的对象在XML中配置，Spring框架读取配置文件来创建对象，用IoC取出。
```XML
<bean class="com.southwind.ioc.Dao">
    <property name="driverName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/testdb"/>
    <property name="username" value="root"/>
    <property name="password" value="123456"/>
</bean>
```
  基于注解：
   1、配置类 
     用Java类替代XML文件，将XML中配置的，放在配置类中。
```JAVA
@Configuration
public class BeanConfiguration {
    
    @Bean(value="可以在这里覆盖方法名") 
    public DataConfig dataConfig() { //方法名就是默认值
        DataConfig dataConfig = new DataConfig();
        dataConfig.setDriverName("Driver");
        dataConfig.setUrl("localhost:3306/dbname");
        dataConfig.setUsername("root");
        dataConfig.setPassword("root");
        return dataConfig;
    }
}

调用：
ApplicationContext context =new AnnotationConfigApplicationContext(上面的类名和方法名/一般直接用包名..basePackage"路径")

ApplicationContext 用来生成IoC
```
  2、扫包+注解
    不依赖于XML或配置类，直接将Bean的创建交给目标类
```JAVA
   直接给对应的抽象类添加
   @Component
   给对应的属性添加
   @Value("对应数据")

   给添加注解的类，添加IoC
   ApplicationContext context =new AnnotationConfigApplicationContext(上面的类名和方法名/一般直接用包名..basePackage"路径")

   @Autowired
   自动装载嵌套的IoC对象，通过类型进行注入
   默认根据byType方式自动装载，增加 @Qualifier("名称")，就会根据byName方式寻找
```


## **1.2 AOP** 
面向切面编程，对面向对象编程的补充，目的是降低耦合度，做到业务代码和非业务代码的解耦合
![[Pasted image 20250701181406.png]]
 1、创建切面类
```Java
  在切面类上面添加@Aspect和@Component
  
  在切面类方法中用 joinPoint 来获取对应实现类的方法的各种数据内容
  添加@Before("execution(属性 返回值类型 目标方法所在的类.对应的方法*(..)") 表明在方法实现前进行
  
  
  添加@AfterReturning(value = "execution(属性 返回值类型 目标方法所在的类.对应的方法*(..)",returning = "result")，
```

 2、添加实现类 在上面添加 @Component
    还需要是继承自implement接口类

 3、配置自动扫包，生成代理配置对象
 ```XML
 <!-- 自动扫包 -->
<context:component-scan base-package="com.southwind.aop">
</context:component-scan>
<!-- 开启自动生成代理 -->
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```


 
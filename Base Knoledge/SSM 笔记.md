核心容器 -> 整合 -> AOP -> 事务


## 核心概念
 - **IOC 控制反转**
    使用对象时，由主动new产生对象转换为外部提供对象，此过程中，对象创建控制权由程序转移到外部。
    Spring技术对IoC思想进行了实现，其提供了一个IoC容器。
    IoC容器负责对象的创建与初始化等一系列工作，被创建或管理的对象叫做**Bean**。
- **DI**依赖注入
    在容器中建立Bean与依赖容器之间的关系

目标：
 - 使用IoC容器管理Bean
 - 在IoC容器内将有依赖关系的Bean进行关系绑定


### **IOC案例分析**
  ![[Pasted image 20250707143028.png|400]]
```XML
<!--首先去POM.XML导入Spring坐标-->
```
![[Pasted image 20250707143943.png|400]]
```XML
<!-- 配置BEAN -->
<bean id="给BEAN起个名字" class = "对应实现类"/>
```
```JAVA
//获取IoC容器
ApplicationContext 名字 = new ClassPathXmlApplicationContext("xml文件名")
```
```JAVA
//获取BEAN
类 对象名字 = (类) 名字.getBean("BEAN的ID")
或者
类 对象名字 = 名字.getBean("BEAN的ID",类.class)
也能直接按类型，但没有意义
```


### **DI案例分析**
![[Pasted image 20250707144221.png|400]]
```JAVA
//删除业务层中使用new方法创建的对象
private 类 类名
```
```XML
<!--配置Service与DAO的关系 -->
<bean id = "" class = "">
<!--property标签表示配置当前bean的属性
name表示配置哪一个具体属性
ref表示参照哪一个Dao-->
    <property name = "业务层中的类名" ref="Beanid里的名字" />
</bean>
```

### **Bean基础配置**
![[Pasted image 20250708185039.png|500]]
name相当于是一个bean的别名
Spring默认生成的Bean是单例，如果要非默认，那就需要增加属性
```XML
<  scope = "prototype"  >
```
有状态的对象不建议交给容器进行管理
![[Pasted image 20250707145452.png|300]]

#### 实例化
- 如果提供了有参构造方法， 但没有写无参构造方法，会抛出异常
- 静态工厂，增加一个factory-method = "对应类的get"，通过工厂实例化Bean，然后在进行转换。
	```JAVA
	<bean id="给BEAN起个名字" class = "工厂类名" factory-method = "工厂内真正造类的方法"/>
	``` 
- 实例工厂与FactoryBean
  ![[Pasted image 20250707150802.png|300]]

#### **BEAN的生命周期**
```XML
init-method ="init" destory-method ="destory"
```
或者
```JAVA
继承接口InitializingBean和DisposableBean
生成方法destory和afterPropertiesSet
```
这样就不需要在Xml中配置，而是直接在对应类中接上接口解决控制生命周期的方法


### **依赖注入**
![[Pasted image 20250708185133.png|400]]
方法：
    普通set
    构造方法
- 包含引用类型和简单类型
```XML
如下均写入对应Bean里
Setter注入 这个最重要
<property name = "业务层中的类名" value="Beanid里的名字" />

构造方法注入
<constructor-agr name="参数名字" ref=""或者value=""/>
下面这么写，耦合程度小，但简单类型的参数顺序不可以变
<constructor-agr type="数据类型" value=""/>
或者按照位置来填入
<constructor-agr index="0" value=""/>
```

#### 自行装配
```XML
按照类型装配需要实现类必须唯一
<bean id="给BEAN起个名字" class = "类" autowire= "byType"/>

按照名称装配则可以Bean去做多个重复实现类,其中class中需要实现的对象的名称需要和对应的Bean名称一样
<bean id="给BEAN起个名字" class = "类" autowire= "byName"/>

集合的注入（简单类型）
<property name="list/set/array">
  <list/set/array>
    <value>itcast</value>
    <value>itheima</value>
    <value>boxuegu</value>
  </list/set/array>
</property>

<property name="map">
  <map>
    <entry key="country" value="china"/>
    <entry key="province" value="henan"/>
    <entry key="city" value="wuhan"/>
  </map>
</property>

<property name="properties">  //props惊险字符串，用于纯配置项
    <props>
        <prop key="country">china</prop>
        <prop key="province">china</prop>
        <prop key="city">china</prop>
    </props>
</property>
```

##### 案例：数据源对象管理
```XML
↓第三方，阿里提供的Druid库，通过缓存池优化数据库与后端数据传送关系
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/> 
    <property name="url" value="jdbc:mysql://localhost:3306/db_name"/>
    <property name="username" value="root"/> 
    <property name="password" value="123456"/> 
</bean>
```
实际上，第三方的东西也是可以通过创建BEAN，实例化，注入这个流程来进行的。

#### 加载外部properties文件
```XML
1、开启context命名空间
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       
    (1)    xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
       
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           
    (2)    http://www.springframework.org/schema/context
    (3)    http://www.springframework.org/schema/context/spring-context.xsd">
2、利用命名空间加载properties文件
<context:property-placeholder location = "classpath:* 对应的配置文件名字">

3、${配置文件.属性名}
```

### 容器
```JAVA
1、加载类路径下的配置文件
ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml")

2、加载文件路径下的配置文件
ApplicationContext ctx = new FileSystemXmlApplicationContext("绝对路径")

```


## 注解开发

### 
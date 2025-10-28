[详解介绍JUnit单元测试框架（完整版） junit安装步骤-CSDN博客](https://blog.csdn.net/qq_26295547/article/details/83145642)
[Junit官网](http://junit.org/)
JUnit 是一个编写可重复测试的简单框架。它是单元测试框架的 xUnit 架构的一个实例。

```maven
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

**JUnit编写单元测试**
创建JunitDemo类，编写第一个单元测试用例
**@Test** 用来注释一个普通的方法为一条测试用例。
**assertEquals()** 方法用于断言两个值是否相关。

**测试功能模块**
```java
public class Count {
    /**
     * 计算并返回两个参数的和
     */
    public int add(int x ,int y){
        return x + y;
    }
}

进行测试
import static org.junit.Assert.assertEquals;
import org.junit.Test;
 
public class CountTest {
    @Test
    public void testAdd() {
        Count count = new Count();
        int result = count.add(2,2);
        assertEquals(result, 4);
    }
}
```

| 注解          | 说明                                                            |
| ----------- | ------------------------------------------------------------- |
| @Test：      | 标识一条测试用例。 (A) (expected=XXEception.class)   (B) (timeout=xxx) |
| @Ignore:    | 忽略的测试用例。                                                      |
| @Before:    | 每一个测试方法之前运行。                                                  |
| @After :    | 每一个测试方法之后运行。                                                  |
| @BefreClass | 所有测试开始之前运行。                                                   |
| @AfterClass | 所有测试结果之后运行。                                                   |
在 testAdd() 用例中设置 **timeout=100** , 说明的用例的运行时间不能超过 100 毫秒。

**JUnit 注解之Fixture**
Test Fixture 是指一个测试运行所需的固定环境
在进行测试时，我们通常需要把环境设置成已知状态（如创建对象、获取资源等）来创建测试，每次测试开始时都处于一个固定的初始状态；测试结果后需要将测试状态还原，所以，测试执行所需要的固定环境称为 Test Fixture。
```java
import static org.junit.Assert.*;
import org.junit.*;
 
public class TestFixture {
    //在当前测试类开始时运行。
    @BeforeClass
    public static void beforeClass(){
        System.out.println("-------------------beforeClass");
    }
 
    //在当前测试类结束时运行。
    @AfterClass
    public static void afterClass(){
        System.out.println("-------------------afterClass");
    }
 
    //每个测试方法运行之前运行
    @Before
    public void before(){
        System.out.println("=====before");
    }
 
    //每个测试方法运行之后运行
    @After
    public void after(){
        System.out.println("=====after");
    }
 
    @Test
    public void testAdd1() {
        int result=new Count().add(5,3);
        assertEquals(8,result);
        System.out.println("test Run testadd1");
    }
 
    @Test
    public void testAdd2() {
        int result=new Count().add(15,13);
        assertEquals(28,result);
        System.out.println("test Run testadd2");
    }
 
}
```

#### @FixMethodOrder
JUnit 通过 **@FixMethodOrder** 注解来控制测试方法的执行顺序的。**@FixMethodOrder** 注解的参数是 **org.junit.runners.MethodSorters** 对象,在枚举类 **org.junit.runners.MethodSorters** 中定义了如下三种顺序类型：
- MethodSorters.JVM
按照JVM得到的方法顺序，也就是代码中定义的方法顺序
- MethodSorters.DEFAULT
以确定但不可预期的顺序执行
- MethodSorters.NAME_ASCENDING
按方法名字母顺序执行

```java
// 按字母顺序执行
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class TestRunSequence {
    @Test
    public void TestCase1() {
        assertEquals(2+2, 4);
    }
    @Test
    public void TestCase2() {
        assertEquals(2+2, 4);
    }
    @Test
    public void TestAa() {
        assertEquals("hello", "hi");
    }
}
```

|方法|说明|
|---|---|
|assertArrayEquals(expecteds, actuals)|查看两个数组是否相等。|
|assertEquals(expected, actual)|查看两个对象是否相等。类似于字符串比较使用的equals()方法。|
|assertNotEquals(first, second)|查看两个对象是否不相等。|
|assertNull(object)|查看对象是否为空。|
|assertNotNull(object)|查看对象是否不为空。|
|assertSame(expected, actual)|查看两个对象的引用是否相等。类似于使用“==”比较两个对象。|
|assertNotSame(unexpected, actual)|查看两个对象的引用是否不相等。类似于使用“!=”比较两个对象。|
|assertTrue(condition)|查看运行结果是否为true。|
|assertFalse(condition)|查看运行结果是否为false。|
|assertThat(actual, matcher)|查看实际值是否满足指定的条件。|
|fail()|让测试失败。|
#### 通过测试套件批量运行
这种方法引入一种 **“测试套件”** 的概念，JUnit 提供了一种批量运行测试类的方法，叫测试套件。
测试套件的写法需要遵循以下原则：
1. 创建一个空类作为测试套件的入口；
2. 使用注解 **org.junit.runner.RunWith** 和 **org.junit.runners.Suite.SuitClasses** 修饰这个空类。
3. 将 **org.junit.runners.Suite** 作为参数传入给注解 **RunWith**，以提示 JUnit 为此类测试使用套件运行器执行。
4. 将需要放入此测试套件的测试类组成数组作为注解 SuiteClasses 的参数。
5. 保证这个空类使用public修饰，而且存在公开的不带任何参数的构造函数。

单独创建一个测试类 runAllTest .
```java
package test;
import org.junit.runner.RunWith;
import org.junit.runners.Suite;
import org.junit.runners.Suite.SuiteClasses;
 
@RunWith(Suite.class)
@SuiteClasses({
        CountTest.class,
        TestFixture.class,
        AssertTest.class,
        TestRunSequence.class,
})
public class runAllTest {
 
}
```
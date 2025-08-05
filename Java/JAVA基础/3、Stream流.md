**结合lambda表达式，简化集合、数组的操作**
1、利用stream流，把数据放上去
2、利用stream流操作数据

| 获取方式         | 方法名                                             | 说明                 |
| ------------ | ----------------------------------------------- | ------------------ |
| ​**​单列集合​**​ | `default Stream<E> stream()`                    | Collection接口中的默认方法 |
| ​**​双列集合​**​ | 需要用.entrySet()进行间接转换，转换为单列集合                    | 无法直接使用stream流      |
| ​**​数组​**​   | `public static <T> Stream<T> stream(T[] array)` | Arrays工具类中的静态方法    |
| ​**​零散数据​**​ | `public static<T> Stream<T> of(T... values)`    | Stream接口中的静态方法     |
```java
list.stream().forEach(s->System.println(s));

map.entrySet().stream().forEach(s->System.prinln(s));
map.keySet().stream().forEach(s->System.prinln(s)); 这个只有键，没有值

Arrays.steam(arr1).forEach(s->System.prinln(s));
```

## Stream流中间方法
| 方法签名                                     | 说明                         |
| ---------------------------------------- | -------------------------- |
| `filter(Predicate<? super T> predicate)` | 过滤,只留下满足条件的数据              |
| `limit(long maxSize)`                    | 获取前几个元素                    |
| `skip(long n)`                           | 跳过前几个元素                    |
| `distinct()`                             | 元素去重，依赖(hashCode和equals方法) |
| `static <T> concat(Stream a, Stream b)`  | 合并a和b两个流为一个流               |
| `map(Function<T, R> mapper)`             | 转换流中的数据类型                  |
```java
public class ObjectFilterExample {
    public static void main(String[] args) {
        List<Product> products = List.of(
            new Product("手机", 3999, "电子"),
            new Product("衬衫", 199, "服装"),
            new Product("笔记本", 5999, "电子"),
            new Product("鞋子", 299, "服装")
        );
        
        // 1. 过滤电子类产品
        List<Product> electronics = products.stream()
                .filter(p -> "电子".equals(p.getCategory()))
                .collect(Collectors.toList());
        electronics.forEach(p -> 
            System.out.println(p.getName() + ": " + p.getPrice()));
        
        // 2. 过滤高价产品(价格>1000)
        List<Product> expensive = products.stream()
                .filter(p -> p.getPrice() > 1000)
                .collect(Collectors.toList());
        
        // 3. 组合条件过滤
        List<Product> result = products.stream()
                .filter(p -> p.getPrice() < 500)
                .filter(p -> "服装".equals(p.getCategory()))
                .collect(Collectors.toList());
    }
}
```
**Map用法——数据转换**
```java
// 将字符串转换为大写
List<String> upperCaseNames2 = names.stream() 
.map(String::toUpperCase) 
.collect(Collectors.toList());

// 每个数字平方
List<Integer> numbers = List.of(1, 2, 3, 4, 5);
List<Integer> squares = numbers.stream()
        .map(n -> n * n)
        .collect(Collectors.toList());

// 提取所有人的名字
List<String> names = people.stream() 
.map(Person::getName) // 方法引用 
.collect(Collectors.toList());
```

## Stream流终结方法
| 方法名称                            | 说明         | 备注           |
| ------------------------------- | ---------- | ------------ |
| `void forEach(Consumer action)` | 遍历流中的每个元素  | 无返回值         |
| `long count()`                  | 统计流中元素的数量  | 返回 `long`类型  |
| `toArray()`                     | 将流数据收集到数组中 | ​**​重点方法​**​ |
| `collect(Collector collector)`  | 将流数据收集到集合中 | ​**​重点方法     |
**匹配检查方法**

| 方法                     | 说明           | 示例                                        |
| ---------------------- | ------------ | ----------------------------------------- |
| `anyMatch(Predicate)`  | 检查是否有元素匹配条件  | `stream.anyMatch(s -> s.length() > 3)`    |
| `allMatch(Predicate)`  | 检查所有元素是否匹配条件 | `stream.allMatch(s -> s.startsWith("A"))` |
| `noneMatch(Predicate)` | 检查是否没有元素匹配条件 | `stream.noneMatch(String::isEmpty)`       |

**查找方法**

| 方法            | 说明              | 示例                   |
| ------------- | --------------- | -------------------- |
| `findFirst()` | 返回第一个元素         | `stream.findFirst()` |
| `findAny()`   | 返回任意一个元素(并行流有用) | `stream.findAny()`   |
**** 归约方法

| 方法                            | 说明         | 示例                               |
| ----------------------------- | ---------- | -------------------------------- |
| `reduce(BinaryOperator)`      | 使用关联函数归约元素 | `stream.reduce((a,b) -> a + b)`  |
| `T reduce(T, BinaryOperator)` | 带初始值的归约    | `stream.reduce(0, Integer::sum)` |

**数值流专用方法**

|方法|说明|示例|
|---|---|---|
|`sum()`|求和(仅限数值流)|`intStream.sum()`|
|`average()`|求平均值|`intStream.average()`|
|`summaryStatistics()`|获取统计摘要|`intStream.summaryStatistics()`|
**** 迭代方法

|方法|说明|示例|
|---|---|---|
|`iterator()`|返回迭代器|`stream.iterator()`|
|`spliterator()`|返回可分割迭代器|`stream.spliterator()`|

## Stream流中 collect方法详解
```JAVA
//按照-分段后选取第二段是否相等
list.stream().filter(s->"男".equals(s.split("-")[1])).collector(Collectors.tolist);

list.stream()
    .filter(s -> "男".equals(s.split(regex:"-")[1]))
    .collect(Collectors.toMap(
        s -> s.split(regex:"-")[0],
        s -> Integer.parseInt(s.split(regex:"-")[2])
    ));
```

在重写 `Collectors.toMap()`的匿名内部类时，​**​只需根据当前生成的是 Key 还是 Value，修改 `Function<T,R>`的第二个泛型参数（`R`）​**​即可。

```java
.collect(Collectors.toMap( 
new Function<流对象, String>() { 
@Override public String apply(String s) { 
return s.split("-")[0]; } 
}, 
new Function<流对象, Integer>() { 
@Override public Integer apply(String s) {
return Integer.parseInt(s.split("-")[2]); 
} 
} )
);
```

## Stream流练习
```java
集合内过滤奇数并保存偶数集合
List<Integer> list = Stream.of(1,2,3,4,5,6,7,8,9,10).filter(s->s%2 == 0).collect(Collectors.toList());

字符串前是姓名，后面是年龄，保留年龄大于24的人，收集到map，名字是键
list.stream().filter(s->Integer.parseInt(s.split(",")[1]>=24)).collect(Collectors.toMap(
    s->s.split(",")[0],
    s->Integer.parseInt(s.split(",")[1]
));
```
![[Pasted image 20250805103034.png]]
```java

```
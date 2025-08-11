![[Pasted image 20250805151434.png]]

# **字节流**
#### **FileOutPutStream**
```
字节输出流的细节：
1. 创建字节输出流对象 
    细节1：参数是字符串表示的路径或者是File对象都是可以的
    细节2：如果文件不存在会创建一个新的文件，但是要保证父级路径是存在的。
    细节3：如果文件已经存在，则会清空文件
2. 写数据
    细节：write方法的参数是整数，但是实际上写到本地文件中的是整数在ASCII上对应的字符
    ‘9’
    ‘7’
3. 释放资源
    每次使用完流之后都要释放资源
```
**创建对象**
```JAVA
throws FileNotFoundException 
FileOutputStream fos =new FileOutputStream("本地文件");
```
**写出数据** **释放资源**
```java
throw IOException
fos.write(97);
fos.close();

byte[] bytes = {97,98,99,100,101};
fos.write(bytes);
fos.write(bytes,索引1，索引2...);
```
```
换行符 \n \r
续写 FileOutputStream fos =new FileOutputStream("本地文件",传递一个TRUE，打开对象时不会清空对应文件);
```

#### **FileInputStream**
```
字节输入流的细节：
1. 创建字节输入流对象
    细节1：如果文件不存在，就直接报错。
2. 写数据
    细节1：一次读一个字节，读出来的是数据在ASCII上对应的数字
    细节2：读到文件末尾了，read方法返回-1。
3. 释放资源
    细节：每次使用完流之后都要释放资源
```
**创建对象**
```java
FileInputStream fos =new FileInputStream("本地文件");
```
**读出数据**
```java
T res = fis.read();
```
**循环读取**
```java
每次使用read都会让指证自动移动一次
while((b = fis.read())!=-1){
    System.output.print((char)b);
}
```

**文件拷贝**
```
1. - 内存占用问题：大文件拷贝时的内存溢出风险
    - 效率问题：传统字节流拷贝的低效性
    - 资源管理：容易忘记关闭流导致资源泄漏
    - 异常处理：拷贝过程中断可能导致数据不一致
2. 可能的解决方案方向：
    - 使用缓冲流(BufferedStream)提升IO效率
    - 采用分块读取/写入机制避免内存溢出
    - try-with-resources语法确保流自动关闭
    - 添加校验机制保证数据完整性
    - 使用NIO的FileChannel进行高效文件传输
```
块拷贝 与 单字节拷贝
```java
- 块拷贝特征：
    • 需要预分配固定内存
    • 但内存开销可通过缓冲区大小调节
- 单字节特征：
    • 几乎不占用额外内存
    • 但无法通过缓冲区提升性能
byte[] bytes = new byte[1024 * 1024 * 5];  // 5MB缓冲区。这里可以动态修改对应buffer的数据
while((len = fis.read(bytes)) != -1) {
    fos.write(bytes, 0, len);  // 批量写入
}

int ch;
while ((ch = fis.read()) != -1) { 
    fos.write(ch);  // 单字节写入
}
```
# **字符流**
#### FileReader
底层逻辑是 **字节流+字符集**
```plaintext
 read() 细节：
 1. read()：默认也是一个字节一个字节的读取的，如果遇到中文就会一次读取多个
 2. 在读取之后，方法的底层还会进行解码并转成十进制
    最终把这个十进制作为返回值
    这个十进制的数据也表示在字符集上的**数字**

 示例说明：
 英文：文件里面二进制数据 01100001
      read() 方法进行读取，解码并转成十进制 97

 中文：文件里面的二进制数据 11100110 10110001 10001001
      read() 方法进行读取，解码并转成十进制 27721
      
如果是非空参，read里可以加入类型后，强制类型转换
read(char)
```
**创建字符输入流对象**
```java
创建字符输入流关联本地文件
FileReader(File file)         
FileReader(String pathname)    

```
**读取数据**
```java
读取数据，读到末尾返回-1,也是默认字节流完成
read();
read(char[] buffer);
```
**释放资源**
```java
关流
close();
```

#### **FileWriter**
```java
基本上，是首先创建一个长度为8192字节数组，作为缓冲区，从文件中读取数据，装满缓冲区，优先读缓冲区数据，没有数据了再返回-1
```

```java
FileWriter(File file) 创建字符输出流关联本地文
FileWriter(String pathname) 创建字符输出流关联本地文件
FileWriter(File file, boolean append) 创建字符输出流关联本地文件,append是续写开关
FileWriter(String pathname, boolean append) 创建字符输出流关联本地文件


void write(int c)   写入单个字符
void write(String str)   写入完整字符串
void write(String str, int off, int len)   写入字符串的指定区间
void write(char[] cbuf)   写入完整字符数组
void write(char[] cbuf, int off, int len)  写入字符数组的指定区间
```
将缓存区文件一口气刷新到本地文件中
```java
flush(); 
```

### **使用场景**：
**字节流**
拷贝任意类型
**字符流**
读取或写出纯文本


## 练习
## 文件加密
![[Pasted image 20250811111453.png|500]]
```java
^利用异或实现加密解密
//创建对象关联文件
FileInputStream fis = new FileInputStream("文件地址");
FileOutputStream fos = new FileOutputStream("文件地址");
//加密处理与资源释放
int b;
while(b = fis.read()!=-1)
{
    fos.write(b^ 加密因子);
}
fos.close();
fis.close();
```

## **修改文件中数据**
![[Pasted image 20250811112131.png|400]]
```java
//也就是取出并进行排序
StringBuilder sb = new StringBuilder();
FileInputStream raw = new FileInputStream("文件");
int b;
while(b = raw.read()!=-1){
    sb.append((char)b);
}
raw.close();

String str = sb.toString();
String[] arrstr = str.split("-");
List<Integer> i = Arrayz.stream(arrstr).map(Integer::parseInt).collect(Collectors.toList);
Collections.sort(i);


FileOutputStream fresh = new FileOutputStream("文件");
String result = i.stream().map(String::valueOf).collect(Collectors.joining("-"));
fresh.write(result.getBytes());
fresh.close();


//更简单一点
Integer[] arr = Arrays.stream(sb.toString)
    .split("-")
    .map(Integer::parseInt)
    .sorted()
    .toArray(Integer[]::new);    
```

# **缓冲流**
![[Pasted image 20250811115128.png]]

#### BufferedInputStream  |  BufferedOutputStream
底层自带了长度8192的缓冲区用来提高性能

```java
利用字节缓冲流拷贝文件
BufferedInputStream bis = new BufferedInputStream(new FileInputStream(文件名，【手动设置缓冲区大小】));
BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(文件名，【手动设置缓冲区大小】));

直接读取并关闭bos就可以了，文件包含关闭基本流的代码。

//2.拷贝(一次读写多个字节)
byte[] bytes = new byte[1024];
int len;
while((len = bis.read(bytes)) != -1){
    bos.write(bytes, 0, len);
}

```

#### BufferedReader  |  BufferedWriter
底层自带了长度8192的缓冲区用来提高性能
```java 
特有方法
- 入
readLine() 读取一行数据或返回null

- 出
newLine() 换行
举例：
bw.write("你嘴角上扬的样子，百度搜索不到");
bw.newLine();
bw.write("以后如果我结婚了，你一定要来哦，没有新娘我会很尴尬");
bw.newLine();
```

### 练习
```java 
修改文本顺序
//重写排序方法
- BufferedReader br = new BufferedReader(new FileReader(文件名));
ArrayList<String> list = new ArrayList<>();
while((line = br.readLine())!=null){
list.add(line);
}
br.close();
Collections.sort(list,new Comparator<String>(){
    @Override
    public int compare(String o1,String o2){
        return  Integer.parseInt(o1.split("\\.")[0])-Integer.parseInt(o2.split("\\.")[0]);
    }
})
BufferedWritter bw = new BufferedWriter(new FileWriter(文件名));
for(String str:list){
bw.write(str);
bw.newLine();
}
bw.close();
//利用TreeMap
BufferedReader br = new BufferedReader(new FileReader("myio\\csb.txt"));
String line;
TreeMap<Integer,String> tm = new TreeMap<>();
while((line = br.readLine())!= null){
    String[] arr = line.split("\\.");
    //0:序号 1:内容
    tm.put(Integer.parseInt(arr[0]),arr[1]);
}
br.close();
```


### 转换流
```java
**场景问题​**​
你有一个字节流（比如从网络收到的数据），但想按​**​文本行​**​读取（字符流操作）：
// 错误！字节流不能直接当字符流用
FileInputStream fis = new FileInputStream("a.txt");
fis.readLine(); // 报错！没有这个方法

// 字节流 → 字符流 → 缓冲流
BufferedReader br = new BufferedReader(
    new InputStreamReader(
        new FileInputStream("a.txt"), "UTF-8" // 指定编码
    )
);
String line = br.readLine(); // 成功读取文本行！
```
### 序列化流
```java
// 1. 必须实现 Serializable 接口（空接口，仅作标记）
class Cat implements Serializable {
    private String name;
    private transient int age; // transient 字段不序列化
}
transient ​：标记的字段不会被序列化

// 2. 序列化（对象 → 文件）
try (ObjectOutputStream oos = new ObjectOutputStream(
        new FileOutputStream("cat.ser"))) {
    oos.writeObject(new Cat("橘座", 3)); // 写入对象
}

// 3. 反序列化（文件 → 对象）
try (ObjectInputStream ois = new ObjectInputStream(
        new FileInputStream("cat.ser"))) {
    Cat cat = (Cat) ois.readObject(); // 强制类型转换

```
 - 使用序列化流将对象写到文件时，需要让Javabean类实现`Serializable`接口，否则会出现`NotSerializableException`异常。
 - 序列化流写到文件中的数据是不能修改的，一旦修改就无法再次读回来了。
 - 序列化对象后，修改了Javabean类，再次反序列化，会出问题，会抛出`InvalidClassException`异常。
 
    若类未显式声明`serialVersionUID`，Java会根据类结构（字段名/类型/方法等）自动生成一个哈希值作为版本号。→ ​**​任何类结构变更都会导致自动生成的版本号改变​**​。
```java
private static final long serialVersionUID = 1L; // 手动固定版本号
```


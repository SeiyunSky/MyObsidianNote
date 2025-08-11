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
# **字符流**：
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








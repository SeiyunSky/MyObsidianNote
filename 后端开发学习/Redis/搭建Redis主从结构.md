1）**创建互不干扰的三个目录
**
2）**修改conf文件**，**恢复原始配置**

3）**利用cp命令拷贝文件进目标目录**
![[Pasted image 20250915224412.png]]

4）**修改每个实例的端口和目录**
`sed -i -e '需要替换的内容' -e '需要替换的内容' 需要替换的文件`
利用linux里的sed替换redis配置

5） **修改每个实例的声明IP**
修改config文件的`replica-announce-ip`

6）**启动redis实例**
**永久生效**
在redis.conf配置内添加`slave of <masterip><masterport>`
**临时生效**
在redis-cli客户端内连接到redis服务，执行命令(也可以用replicaof)

**Redi主从同步——repl_baklog的offset**

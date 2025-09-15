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


## 引入哨兵集群
1）创建多个哨兵目录，创建sentinel.conf文件
```ini
port 27001    //哨兵端口
sentinel announce-ip 192.168.150.101    //哨兵声明IP
sentinel monitor mymaster 192.168.150.101 7001 2    //监视的主机  2指的是quorum人数
sentinel down-after-milliseconds mymaster 5000    //slave与master断开超时时间
sentinel failover-timeout mymaster 60000  //s和m恢复最长时间
dir "/tmp/s1"
```


## RedisTemplate的哨兵模式
在Sentinel集群监管下的Redis主从集群会因为自动故障转移发生变化，Spring的RedisTemplate利用lettuce实现了节点的感知和自动切换。
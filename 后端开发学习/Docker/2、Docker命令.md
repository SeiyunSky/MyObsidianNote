### 服务相关命令
```linux
启动docker
systemctl start docker
关闭docker
systemctl status docker
重启docker
systemctl restart docker
开机启动docker
systemctl enable docker
```
### 镜像相关命令
```docker
查看本地镜像
docker images

搜索对应的镜像文件
docker search 想下载的镜像

下载对应的文件
docker pull redis:5.0

删除对应的文件(版本号或ImageID)
docker rmi redis:latest
```

**容器相关命令**
```docker
创建容器
-i 保持容器一直允许
-t 创建交互式容器  -d 后台运行终端 守护式容器
--name= 起名字
中间放版本号和版本名
/bin/bash 进入docker容器内部
-p 宿主机端口:容器内端口
-e 设置环境变量
docker run -i -t --name=c1 centos:7 /bin/bash

退出容器 回到宿主机
exit 

查看正在运行容器
-a 查看全部已创建容器
-q 只获取容器id
docker ps 

停止容器
docker stop name

启动容器
docker start name

删除容器
docker rm 'docker ps -aq'
docker rm id|name

查看容器消息
docker inspect name
```


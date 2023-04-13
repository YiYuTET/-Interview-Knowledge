# CentOS7---ZooKeeper伪分布式安装部署



## 1. 下载ZooKeeper安装包：[下载地址](https://www.apache.org/dyn/closer.cgi/zookeeper)

注意，随着版本的更新，3.5版本以后的压缩包分成了两种
带bin的压缩包是真正的标准压缩包
而不带bin的压缩包是源码压缩包
我们需要使用文件名带有bin 的那个压缩包，例如：apache-zookeeper-3.6.3-bin.tar.gz 这样解压后才会有lib目录下的那些jar包。


## 2. 解压安装包

这里我解压到/home/software目录下

```shell
tar -zxvf apache-zookeeper-3.6.3-bin.tar.gz  -C /home/software/
```


## 3. 复制oo_example.cfg文件并改名
进入解压后的路径zookeeper-3.4.10/conf路径下，复制zoo_example.cfg配置文件，因为是伪分布式安装，一台机器上有3个zookeeper进程，所以这里复制3份。
```shell
cp zoo_example.cfg zoo1.cfg
cp zoo_example.cfg zoo2.cfg
cp zoo_example.cfg zoo3.cfg
```


## 4.分别输入如下配置
## zoo1.cfg
```shell
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/software/apache-zookeeper-3.6.3-bin/dataDir1
clientPort=2181
server.1=localhost:2887:3887
server.2=localhost:2888:3888
server.3=localhost:2889:3889
```

## zoo2.cfg
```shell
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/software/apache-zookeeper-3.6.3-bin/dataDir2
clientPort=2182
server.1=localhost:2887:3887
server.2=localhost:2888:3888
server.3=localhost:2889:3889
```
## zoo3.cfg
```shell
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/software/apache-zookeeper-3.6.3-bin/dataDir3
clientPort=2183
server.1=localhost:2887:3887
server.2=localhost:2888:3888
server.3=localhost:2889:3889
```
tickTime - 心跳的时间ms
initLimit - 初始化限制时间
syncLimit - 同步限制时间
dataDir - 数据存放物理路径
clientPort - 供客户端连接的端口
server.n=host:mainPort:selectPort -- 表示集群的配置，其中n表示集群中的n个节点，host表示集群对应的ip，mainPort -- 表示该节点作为主节点对应的端口，selectPort -- 表示集群主节点选举时彼此通信的端口

## 5.新建对应的dataDir文件夹和myid文件

在ZooKeeper根目录下，新建dataDir文件夹，分别是zoo1，zoo2，zoo3的dataDir路径
```shell
mkdir dataDir1
mkdir dataDir2
mkdir dataDir3
```
然后分别在每个dataDir中创建myid文件并写入zoo1，zoo2，zoo3的id
```shell
echo 1 >>  dataDir1/myid
echo 2 >>  dataDir2/myid
echo 3 >>  dataDir3/myid
```


## 6. 启动
进入zookeeper的bin目录下，通过脚本文件zkServer.sh启动
```shell
./zkServer.sh start ../conf/zoo1.cfg
./zkServer.sh start ../conf/zoo2.cfg
./zkServer.sh start ../conf/zoo3.cfg
```

查看每个zoo的状态
```shell
./zkServer.sh status ../conf/zoo1.cfg
./zkServer.sh status ../conf/zoo2.cfg
./zkServer.sh status ../conf/zoo3.cfg
```


通过集群暴露的客户端连接接口，可以直接访问集群。
```shell
./zkCli.sh -server localhost:2181
```
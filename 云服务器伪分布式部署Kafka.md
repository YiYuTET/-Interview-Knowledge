# 云服务器伪分布式部署kafka

## 1.前言

本次安装的kafka使用了独立的zookeeper，所以需先搭建好zookeeper集群。

[ZooKeeper伪分布式安装部署](https://yiyuyyds.cn/archives/centos7---zookeeper%E4%BC%AA%E5%88%86%E5%B8%83%E5%BC%8F%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2)

[Kafka下载地址](https://kafka.apache.org/downloads)

## 2.解压安装包

这里我解压到/home/software目录下
```shell
tar -zxvf kafka_2.12-3.0.0.tgz -C /home/software/
```

## 3.配置环境变量
```shell
vim /etc/profile
```

```shell
export KAFKA_HOME=/home/software/kafka_2.12-3.0.0
export PATH=$PATH:/$KAFKA_HOME/bin
```
```shell
source /etc/profile
```

## 4.复制server.properties文件并改名
进入kafka的config路径下，复制server.properties配置文件，因为是伪分布式安装，一台机器上有3个kafka进程，所以这里复制3份。

```shell
cp server.properties server_1.properties
cp server.properties server_2.properties
cp server.properties server_3.properties
```


## 5.分别输入如下配置

### server_1.properties

```shell
broker.id=0
log.dirs=/home/software/kafka_2.12-3.0.0/logs/log_server_1
listeners=PLAINTEXT://0.0.0.0:9092
advertised.listeners=PLAINTEXT://114.116.24.98:9092
zookeeper.connect=192.168.0.219:2181
```

### server_2.properties

```shell
broker.id=1
log.dirs=/home/software/kafka_2.12-3.0.0/logs/log_server_2
listeners=PLAINTEXT://0.0.0.0:9093
advertised.listeners=PLAINTEXT://114.116.24.98:9093
zookeeper.connect=192.168.0.219:2181
```


### server_3.properties


```shell
broker.id=2
log.dirs=/home/software/kafka_2.12-3.0.0/logs/log_server_3
listeners=PLAINTEXT://0.0.0.0:9094
advertised.listeners=PLAINTEXT://114.116.24.98:9094
zookeeper.connect=192.168.0.219:2181
```


## 6.启动

进入kafka的bin目录下，运行
```shell
kafka-server-start.sh -daemon ../config/server_1.properties
kafka-server-start.sh -daemon ../config/server_2.properties
kafka-server-start.sh -daemon ../config/server_3.properties
```

通过jps命令可以查看3个kafka进程说明部署成功。
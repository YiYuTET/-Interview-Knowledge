# 1. Kafka 基本概念

## 1.1 什么是kafka

<font size=3><b>Kafka最初是由LinkedIn即领英公司基于Scala和Java语言开发的分布式消息发布-订阅系统，现已捐献给Apache软件基金会。其具有高吞吐、低延迟的特性，许多大数据实时流式处理系统比如Storm、Spark、Flink等都能很好的与之集成。
</b></font>

<font size=3><b>总的来讲，Kafka通常具有3重角色：</b></font>

- <font size=3><b>存储系统：通常消息队列会把消息持久化到磁盘，防止消息丢失，保证消息可靠性。Kafka的消息持久化机制和多副本机制使其能够作为通用数据存储系统使用。</b></font>
- <font size=3><b>消息系统：Kafka和传统的消息队列比如RabbitMQ、RocketMQ、ActiveMQ类似，支持流量削峰、服务解耦、异步通信等核心功能。</b></font>
- <font size=3><b>流处理平台：Kafka不仅能够与大多数流式计算框架完美整合，并且自身也提供了一个完整的流式处理库，即Kafka Streaming。Kafka Streaming提供了类似Flink中的窗口、聚合、变换、连接等功能。
</b></font>

<font size=3><b><u>一句话概括：Kafka是一个分布式的基于发布/订阅模式的消息队列(Message Queue)，在业界主要应用于大数据实时流式计算领域。</u>
</b></font>

## 1.2 kafka的特点

- <font size=3><b>高吞吐、低延迟：kafka每秒可以处理几十万条消息，它的延迟最低只有几毫秒，每个topic可以分多个partition，由多个consumer group对partition进行consume操作。</b></font>
- <font size=3><b>可扩展性：kafka集群支持热扩展</b></font>
- <font size=3><b>持久性、可靠性：消息被持久化到本地磁盘，并且支持数据备份防止数据丢失</b></font>
- <font size=3><b>容错性：允许集群中有节点失败(若副本数量为n，则允许n-1个节点失败)</b></font>
- <font size=3><b>高并发：支持数千个客户端同时写入
</b></font>

<font size=3><b>Kafka在各种应用场景中，起到的作用可以归纳为这么几个术语：削峰填谷、解耦！在大数据流式计算场景领域中，kafka主要作为计算系统的前置缓存和输出结果缓存。
</b></font>

# 2. 安装部署

- 上传安装包
- 解压
- 修改配置文件
(1)进入配置文件系统
```shell
[root@doit01 apps]# cd kafka_2.12-2.3.1/config
```
(2)编辑配置文件
```properties
# 为依次增长的：0、1、2、3、4，集群中唯一id
broker.id=0

# 数据存储的目录
log.dirs=/opt/apps/data/kafkadata

# 底层存储的数据(日志)留存时长(默认7天)
log.retention.hours=168

# 底层存储的数据(日志)留存量(默认1G)
log.retention.bytes=1073741824

# 指定zk集群地址
zookeeper.connect=doit01:2181,doit02:2181,doit03:2181
```

- 分发安装包

```sh
for i in {2..3}
do
scp -r kafka_2.11-2.2.2 linux0$i:$PWD
done
```
安装包分发后，记得修改broker.id

- 配置环境变量
- 启停集群(在各个节点上启动)
```shell
bin/kafka-server-start.sh -daemon /opt/apps/kafka_2.11-2.2.2/config/server.properties

# 停止集群
bin/kafka-server-stop.sh stop
```



# 3. Kafka运维监控

<font color=red size=3><b>详见 https://www.bilibili.com/video/BV1Xr4y1t7mQ?p=6&vd_source=dd05d982b6c8a7e631e7f07548a539b1
</b></font>
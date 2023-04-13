# 8. Kafka 系统架构

![Kafka图2](https://cdn.jsdelivr.net/gh/YiYuTET/ImageStorage/202304132051118.png)

<font size=3><b>自我推导设计：</b></font>

- <font size=3><b>kafka是用来存数据的</b></font>
- <font size=3><b>现实世界数据有分类，所以存储系统也应有数据分类管理功能，如mysql的表；kafka有topic</b></font>
- <font size=3><b>如一个topic的数据全部交给一台server存储和管理，则读写吞吐量有限</b></font>
- <font size=3><b>所以，一个topic的数据应该可以分成多个部分(partition)分别交给多台server存储和管理</b></font>
- <font size=3><b>如果一台server宕机，这台server负责的partition将不可用，所以，一个partition应有多个副本</b></font>
- <font size=3><b>一个partition有多个副本，则副本间的数据一致性难以保证，因此要有一个leader统领读写</b></font>
- <font size=3><b>一个leader万一挂掉，则该partition又不可用，因此还要有leader的动态选举机制</b></font>
- <font size=3><b>集群有哪些topic，topic有哪几个分区，server在线情况，等等元信息和状态信息需要在集群内部及客户端之间共享，则引入了zookeeper</b></font>
- <font size=3><b>客户端在读取数据时，往往需要知道自己所读取的位置，因而要引入消息偏移量维护机制
</b></font>



## 8.1 broker服务器

<font size=3><b>一台kafka服务器就是一个broker，一个kafka集群由多个broker组成。
</b></font>



## 8.2 生产者producer

<font size=3><b>消息生产者，就是向kafka broker发消息的客户端。
</b></font>


## 8.3 消费者consumer

<font size=3><b>consumer：消费者，从kafka broker取消息的客户端。
consuemr group：消费组，单个或多个consumer可以组成一个消费组。
</b></font>



## 8.4 主题Topic与分区Partition

<font size=3><b>在 Kafka 中消息是以 Topic 为单位进行归类的，Topic 在逻辑上可以被认为是一个 Queue，Producer 生产的每一条消息都必须指定一个 Topic，然后 Consumer 会根据订阅的 Topic 到对应的 broker 上去拉取消息。</b></font>

<font size=3><b>为了提升整个集群的吞吐量，Topic 在物理上还可以细分多个分区，一个分区在磁盘上对应一个文件夹。由于一个分区只属于一个主题，很多时候也会被叫做主题分区(Topic-Partition)。
</b></font>


## 8.5 分区副本replica

<font size=3><b>每个topic的每个partition都可以配置多个副本(replica)，以提高数据的可靠性；
每个partition的所有副本中，必有一个leader副本，其它的就是follow副本(observer副本)；
follow定期找leader同步最新的数据，对外提供服务只有leader；
</b></font>


## 8.6 分区副本leader

<font size=3><b>partition replica中的一个角色，在一个partition的多个副本中，会存在一个副本角色为leader；
producer和consumer只能跟leader交互(读写数据)；
</b></font>


## 8.7 分区follower

<font size=3><b>partition replica中的一个角色，它通过心跳通信不断从leader中拉取、复制数据(只负责备份)。如果leader所在节点宕机，follower中会选举出新的leader；
</b></font>


## 8.8 消息偏移量offset

<font size=3><b>partition中每条消息都会被分配一个递增id(offset)，通过offset可以快速定位到消息的存储位置；kafka只保证按一个partition中的消息的顺序，不保证一个topic整体(多个partition间)的顺序。
</b></font>

![Kafka图3](https://cdn.jsdelivr.net/gh/YiYuTET/ImageStorage/202304132051824.jpeg)





# 9. Kafka的数据存储结构

# 9.1 Kafka的整体存储结构

![Kafka图5](https://cdn.jsdelivr.net/gh/YiYuTET/ImageStorage/202304132051828.jpg)

# 9.2 物理存储目录结构

- <font size=3><b>存储目录 名称规范：topic名称-分区号   
例如：t1-0、t1-1  
"t1"即为一个topic的名称；    
而"t1-0/t1-1"则表明这个目录是t1这个topic的哪个partition
</b></font>

- <font size=3><b>数据文件 名称规范    
生产者生产的消息会不断追加到log文件末尾，为防止log文件过大导致数据定位效率低，kafka采取了分片和索引机制；    
每个partition的数据将分为多个segment存储     
每个segment对应两个文件，“.index文件”和“.log文件”    
index和log文件以当前segment的第一条消息的offset命名。
</b></font>

![Kafka图4](https://cdn.jsdelivr.net/gh/YiYuTET/ImageStorage/202304132051462.jpeg)

<font size=3><b>index索引文件中的数据为：消息offset -> log文件中该消息的物理偏移量位置；</b></font>

<font size=3><b>Kakfa中的索引文件以稀疏索引(sparse index)的方式构造消息的索引，它并不保证每个消息在索引文件中都有对应的索引；每当写入一定量(由broker端参数log.index.interval.bytes指定，默认值为4096，即4KB)的消息时，偏移量索引文件和时间戳索引文件分别增加一个偏移量索引项和时间戳索引项，增大或减小log.index.interval.bytes的值，对应的可以缩小或增加索引项的密度；   
查询指定偏移量时，使用二分查找法来快速定位偏移量的位置。
</b></font>



# 9.3 消息的message存储结构

<font size=3><b>在客户端编程代码中，消息的封装有两种：ProducerRecord、ConsumerRecord
简单来说，++kafka中的每个message由一对key-value构成；++
Kafka中的message格式经历了3个版本的变化：v0、v1、v2
</b></font>

<table><tr><td bgcolor=Gainsboro><font size=4><b>v0</td></tr></table>

![Kafka图6](https://cdn.jsdelivr.net/gh/YiYuTET/ImageStorage/202304132051939.jpg)

<table><tr><td bgcolor=Gainsboro><font size=4><b>v1</td></tr></table>

<font size=3><b>各个字段的含义介绍如下：</b></font>

- <font size=3><b>crc：占用4个字节，主要用于校验消息的内容；</b></font>
- <font size=3><b>magic：这个占用1个字节，主要用于标识Kafka版本。Kafka 0.10.x magic默认值为1</b></font>
- <font size=3><b>attributes：占用1个字节，这里面存储了消息压缩使用的编码以及Timestamp类型。目前Kafka支持gzip、snappy以及lz4(0.8.2引入)三种压缩格式；后四位如果是0001则表示gzip压缩，如果是0010则是snappy压缩，如果是0011则是lz4压缩，如果是0000则表示没有使用压缩。第4个bit位如果为0，代表使用create time；如果为1代表append time；其余位(第5~8位)保留</b></font>
- <font size=3><b>key length：占用4个字节，主要标识Key的内容的长度；</b></font>
- <font size=3><b>key：占用N个字节，存储的是key的具体内容；</b></font>
- <font size=3><b>value length：占用4个字节，主要标识value的内容的长度；</b></font>
- <font size=3><b>value：value即是消息的真实内容，在Kafka中这个也叫做payload。
</b></font>

![Kafka图7](https://cdn.jsdelivr.net/gh/YiYuTET/ImageStorage/202304132051829.jpg)

<table><tr><td bgcolor=Gainsboro><font size=4><b>v2</td></tr></table>

![Kafka图8](https://cdn.jsdelivr.net/gh/YiYuTET/ImageStorage/202304132051007.jpg)
# 5. Kafka 生产者API

<font size=3><b>一个正常的生产逻辑需要具备以下几个步骤：   
（1）配置生产者参数及创建相应的生产者实例     
（2）构建待发送的消息         
（3）发送消息     
（4）关闭生产者实例
</b></font>

<font size=3><b>首先引入maven依赖
</b></font>

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-clients</artifactId>
        <version>2.4.1</version>
    </dependency>
</dependencies>
```

```java
public class ProducerDemo {
    public static void main(String[] args) throws InterruptedException {

        //泛型K：要发送的数据中的key
        //泛型V：要发送的数据中的value
        //隐含之意：kafka中的message，是key-value结构的（可以没有key）
        Properties props = new Properties();
        props.setProperty("bootstrap.servers", "doit01:9092,doit02:9092");

        //因为kafka底层的存储是没有类型维护机制的，用户所发的所有数据类型，都必须变成序列化后的byte[]
        //所以，kafka的producer需要一个针对用户要发送的数据类型的序列化工具
        //且这个序列化工具类，需要实现kafka所提供的序列工具接口，org.apache.kafka.common.serialization.Serializer
        props.setProperty("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.setProperty("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        /**
         * 代码中进行客户端参数配置的另一种写法
         */
        props.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "doit01:9092,doit02:9092");
        props.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.setProperty(ProducerConfig.ACKS_CONFIG, "all");

        //构造一个生产者客户端
        KafkaProducer<String, String> producer = new KafkaProducer<>(props);

        for (int i = 0; i < 100; i++) {
            //将业务数据封装成客户端所能发送的封装格式
            //0->abc0
            //1->abc1
            ProducerRecord<String, String> message = new ProducerRecord<>("abcx", i + "", "abc" + i);

            //调用客户端去发送
            //数据的发送动作在producer的底层是异步线程去异步发送的
            producer.send(message);

            Thread.sleep(100);
        }

        //关闭客户端
        //producer.flush();
        producer.close();
    }
}
```

<font size=3><b>消息对象ProducerRecord，除了包含业务数据外，还包含了多个属性：
</b></font>

```java
public class ProducerRecord<K, V> {
    private final String topic;
    private final Integer partition;
    private final Headers headers;
    private final K key;
    private final V value;
    private final Long timestamp;
```



# 6. Kafka 消费者API

## 6.1 Kafka 消费者API示例

<font size=3><b>一个正常的消费逻辑需要具备以下几个步骤：    
（1）配置消费者客户端参数及创建相应的消费者实例     
（2）订阅主题topic    
（3）拉取消息并消费    
（4）定期向__consumer_offsets主题提交消费位移offset     
（5）关闭消费者实例
</b></font>

```java
public class ConsumerDemo {
    public static void main(String[] args) {

        //构建一个properties来存放消费者客户端参数
        Properties props = new Properties();
        props.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "doit01:9092");
        props.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        props.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

        //kafka的消费者，默认是从所属组之前所记录的偏移量开始消费，如果找不到之前的记录的偏移量，则从如下参数配置的策略来确定消费起始位移
        //可以选择：earliest（自动重置到每个分区的最前一条消息），latest（自动重置到每个分区的最新一条消息），none（没有重置策略）
        props.setProperty(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "latest");

        //设置消费者所属的组id
        props.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "d30-1");

        //设置自动提交最新的消费位移，默认是开启的
        props.setProperty(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");

        //自动提交最新消费位移的时间间隔，默认值就是5000ms
        props.setProperty(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "5000");

        //构造一个消费者客户端
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

        //订阅主题（可以是多个）
        consumer.subscribe(Collections.singletonList("abcx"));

        //显示指定消费起始偏移量
        TopicPartition abcxP0 = new TopicPartition("abcx", 0);
        TopicPartition abcxP1 = new TopicPartition("abcx", 1);
        consumer.seek(abcxP0, 10);
        consumer.seek(abcxP1, 15);


        //循环往复拉取数据
        boolean condition = true;
        while (condition) {
            //客户端去拉取数据的时候，如果服务器没有数据响应，会保持连接等待服务端响应
            //poll中传入的超时时长参数，是指等待的最大时长
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMinutes(Long.MAX_VALUE));

            for (ConsumerRecord<String, String> record : records) {
                //ConsumerRecord中，不光有用户的业务数据，还有kafka塞入的元数据
                String key = record.key();
                String value = record.value();

                //本条数据所属的topic
                String topic = record.topic();
                //本条数据所属的分区
                int partition = record.partition();
                //本条数据的offset
                long offset = record.offset();
                //当前这条数据所在分区的leader的朝代纪元
                Optional<Integer> leaderEpoch = record.leaderEpoch();

                //在kafka的数据底层存储中，不光有用户的业务数据，还有大量元数据
                //timestamp就是其中只一，记录本条数据的时间戳
                //但是时间戳有两种类型，本条数据的创建时间（生产者），本条数据的追加时间（borker写入log文件的时间）
                TimestampType timestampType = record.timestampType();
                long timestamp = record.timestamp();

                //数据头是生产者在写入数据时附加进去的（相当于用户自己自定义的元数据）
                Headers headers = record.headers();

                System.out.println(String.format("数据key：%s, 数据value：%s, 数据所属的topic：%s, 数据所属的partition：%d," +
                                "数据的offset：%d, 数据所属leader的纪元：%s, 数据时间戳类型：%s, 数据的时间戳：%d,",
                        key, value, topic, partition, offset, leaderEpoch.toString(), timestampType.name(), timestamp));
            }
        }

        //关闭客户端
        consumer.close();
    }
}
```


## 6.2 subscribe订阅主题

<font size=3><b>subscribe有如下重载方法：</b></font>

- <font size=3><b>public void subscribe(Collection<String> topics, ConsumerRebalanceListener listener)</b></font>
- <font size=3><b>public void subscribe(Collection<String> topics)</b></font>
- <font size=3><b>public void subscribe(Pattern pattern, ConsumerRebalanceListener listener)</b></font>
- <font size=3><b>public void subscribe(Pattern pattern)
</b></font>

1. <font size=3><b>指定集合方式订阅主题：   
  consumer.subscribe(Arrays.asList(topic1));</b></font>
2. <font size=3><b>正则方式订阅主题   
	如果消费者采用的是正则表达式的方式(subscribe(Pattern))订阅，在之后的过程中，如果有人又创建了新的主题，并且主题名字与正则表达式相匹配，那么这个消费者就可以消费到新添加的主题中的消息。如果应用程序需要消费多个主题，并且可以处理不同的类型，那么这种订阅方式就很有效。
	正则表达式的方式订阅的示例如下   
	consumer.subscribe(Pattern.compile("topic.*"));
	利用正则表达式订阅主题，可实现动态订阅。</b></font>

## 6.3 assign订阅主题

<font size=3><b>消费者不仅可以通过KafkaConsumer.subscribe()方法订阅主题，还可以直接订阅某些主题的指定分区；  
在KafkaConsumer中提供了assign()方法来实现这些功能，此方法的具体定义如下:     
public void assign(Collection<TopicPartition> partitions)    
这个方法只接受参数partitions，用来指定需要订阅的分区集合，示例如下：
</b></font>

```java
consumer.assign(Arrays.asList(new TopicPartition("tpc_1", 0), new TopicPartition("tpc_2", 1)));
```



## 6.4 subscribe和assign的区别

- <font color=red size=3><b>通过subscribe()方法订阅主题具有消费者自动再均衡功能；
</b></font>

<font size=3><b>在多个消费者的情况下可以根据分区分配策略来自动分配各个消费者与分区的关系。当消费组的消费者增加或减少时，分区分配关系会自动调整，以实现消费负载均衡及故障自动转移。
</b></font>

- <font color=red size=3><b>assign()方法订阅分区时，是不具备消费者自动均衡功能的；
</b></font>

<font size=3><b>其实这一点从assign()方法参数可以看出端倪，两种类型subscribe()都有ConsumerRebalanceListener类型参数的方法，而assign()方法却没有。
</b></font>




## 6.5 取消订阅

<font size=3><b>既然有订阅，那么就有取消订阅。   
可以使用KafkaConsumer中的unsubscribe()方法取消主题的订阅，这个方法即可以取消通过subscribe(Collection)方式实现的订阅；   
也可以取消通过subscribe(Pattern)方式实现的订阅，还可以取消通过assign(Collection)方式实现的订阅，示例如下：    
consumer.unsubscribe();     
如果将subscribe(Collection)或assign(Collection)集合参数设置为空集合，作用与unsubscribe()方法相同，如下示例中三行代码的效果相同：
</b></font>

```java
consumer.unsubscribe();
consumer.subscribe(new ArrayList<String>());
consumer.assign(new ArrayList<TopicPartition>());
```


## 6.6 消息的消费模式

<font size=3><b><u>Kafka中的消费是基于拉取模式的。</u></b></font>
<font size=3><b>消息的消费一般有两种模式：推送模式和拉取模式。推送模式是服务端主动将消息推送给消费者，而拉取模式消费者主动向服务端发起请求来拉取消息。</b></font>

<font size=3><b>Kafka中的消息消费是一个不断轮询的过程，消费者所要做的就是重复的调用poll()方法，poll()方法返回的是所订阅的主题（分区）上的一组消息。   
对于poll()方法而言，如果某些分区中没有可供消费的消息，那么此分区对应的消息拉取的结果就为空，如果订阅的所有分区中都没有可供消费的消息，那么poll()方法返回为空的信息集；</b></font>

<font size=3><b>poll()方法具体定义如下:     
public ConsumerRecords<K, V> poll(final Duration timeout)     
超过时间参数timeout，用来控制poll()方法的阻塞时间，在消费者的缓冲区里没有可用数据时会发生阻塞，如果消费者程序只用来单纯拉取并消费数据，则为了提高吞吐率，可以把timeout设置为Long.MAX_VALUE
</b></font>



## 6.7 自动提交消费者偏移量

<font size=3><b>Kafka中默认的消费位移的提交方式是自动提交，这个由消费者客户端参数<u>enable.auto.commit</u>配置，默认值为true。当然这个默认的自动提交不是每消费一条消息就提交一次，而是定期提交，这个定期的周期时间由客户端参数<u>auto.commit.interval.ms</u>配置，默认值为5秒，此参数生效的前提是enable.auto.commit 参数为true。</b></font>

<font size=3><b>在默认的方式下，消费者每隔5秒会将拉取到的每个分区中最大的消息位移进行提交。自动位移提交的动作是在poll()方法的逻辑里完成的，在每次真正向服务端发起拉取请求之前会检查是否可以进行位移提交，如果可以，那么就会提交上一次轮询的位移。</b></font>

<font size=3><b>Kafka消费的编程逻辑中位移提交是一大难点，自动提交消费位移的方式非常简便，它免去了复杂的位移提交逻辑，让编码更简洁。但随之而来的是重复消费和消息丢失的问题。</b></font>

- <font size=3><b>重复消费   
假设刚刚提交完一次消费位移，然后拉取一批消息进行消费，在下一次自动提交消费位移之前，消费者崩溃了，那么又得从上一次位移提交的地方重新开始消费，这样便发生了重复消费的现象（对于再均衡的情况同样适用)。我们可以通过减小位移提交的时间间隔来减小重复消息的窗口大小，但这样并不能避免重复消费的发送，而且也会使位移提交更加频繁。</b></font>

- <font size=3><b>丢失消息    
按照一般思维逻辑而言，自动提交是延时提交，重复消费可以理解，那么消息丢失又是在什么情形下会发生的呢？我们来看下图中的情形：   
拉取线程不断地拉取消息并存入本地缓存，比如在BlockingQueue中，另一个处理线程从缓存中读取消息并进行相应的逻辑处理。设目前进行到了第y+1次拉取，以及第m次位移提交的时候，也就是x+6之前的位移己经确认提交了，处理线程却还正在处理x+3的消息；此时如果处理线程发生了异常，待其恢复之后会从第m次位移提交处，也就是x+6的位置开始拉取消息，那么x+3至x+6之间的消息就没有得到相应的处理，这样便发生消息丢失的现象。
</b></font>

![Kafka图10](https://cdn.jsdelivr.net/gh/YiYuTET/ImageStorage/202304132042073.png)




## 6.8 手动提交消费者偏移量(调用kafka api)

<font size=3><b>自动位移提交的方式在正常情况下不会发生消息丢失或重复消费的现象，但是在编程的世界里异常无可避免；同时，自动位移提交也无法做到精确的位移管理。在Kafka中还提供了手动位移提交的方式，这样可以使得开发人员对消费位移的管理控制更加灵活。</b></font>

<font size=3><b>很多时候并不是说拉取到消息就算消费完成，而是需要将消息写入数据库、写入本地缓存，或者是更加复杂的业务处理。在这些场景下，所有的业务处理完成才能认为消息被成功消费；</b></font>

<font size=3><b>手动的提交方式可以让开发人员根据程序的逻辑在合适的时机进行位移提交。开启手动提交功能的前
提是消费者客户端参数++enable.auto.commit++配置为false，示例如下：
</b></font>

```java
props.put(ConsumerConf.ENABLE_AUTO_COMMIT_CONFIG, false);
```

<font size=3><b>手动提交可以细分为同步提交和异步提交，对应于KafkaConsumer中的commitSync()和commitAsync()两种类型的方法；
</b></font>

- <font size=3><b>同步提交的方式    
commitSync()方法的定义如下：
</b></font>

```java
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
    for (ConsumerRecord<String, String> r : records) {
    //do something to process record.
    }
    consumer.commitSync();
}
```

<font size=3><b>对于采用commitSync()的无参方法，它提交消费位移的频率和拉取批次消息、处理批次消息的频率是一样的，如果想寻求更细粒度的、更精准的提交，那么就需要使用commitSync()的另一个有参方法，具体定义如下：
</b></font>

```java
public void commitSync(final Map<TopicPartition, offsetAndMetadata> offsets)
```

<font size=3><b>示例代码如下：
</b></font>


```java
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
    for (ConsumerRecord<String, String>r : records)(
        long offset = r.offset();
        //do something to process record.
        TopicPartition topicPartition = new TopicPartition(r.topic(), r.partition());
        consumer.commitsync(Collections.singletonMap(topicPartition, new offsetAndMetadata (offset+1)));
    }
}
```

<font color=red size=3><b>提交的偏移量 = 消费完的record的偏移量 + 1
因为，__consumer_offsets中记录的消费偏移量，代表的是，消费者下一次要读取的位置！！!
</b></font>

- <font size=3><b>异步提交的方式   
异步提交的方式(commitAsync())在执行的时候消费者线程不会被阻塞；可能在提交消费位移的结果还未返回之前就开始了新一次的拉取。异步提交可以让消费者的性能得到一定的增强。    
commitAsync方法有一个不同的重载方法，具体定义如下：
</b></font>

```java
public void commitAsync()
public void commitAsync(OffsetCommitCallback callback)
public void commitAsync(final Map<TopicPartition, OffsetAndMetadata> offsets, offsetCommitCallback callback)
```

<font size=3><b>示例代码如下：
</b></font>

```java
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
    for (ConsumerRecord<String, String> r : records) {
        long offset = r.offset ()
        //do something to process record.
        TopicPartition topicPartition = new TopicPartition(r.topic(), r.partition());
        consumer.commitSync(Collections.singletonMap(topicPartition, new OffsetAndMetadata(offset+1)));
        consumer.commitAsync(Collections.singletonMap(topicPartition, new offsetAndMetadata(offset+1)), new offsetcommitcallback() {
        @override
        public void onComplete(Map<TopicPartition, offsetAndMetadata> map, Exception e) {
            if(e == null ) {
                System.out.printIn (map);
            }else{
                System.out.println ("error commit offset");
            }
        }
    });
}
}
```




## 6.9 消费者提交偏移量方式的总结

<font size=3><b>consumer的消费位移提交方式：</b></font>

1. <font size=3><b>全自动 auto.offset..commit = true -> 定时提交到consumer_offsets</b></font>
2. <font size=3><b>半自动 auto.offset.commit = false；然后手动触发提交consumer.commitSync()； -> 提交到consumer_offsets</b></font>
3. <font size=3><b>全手动 auto.offset.commit = false；写自己的代码去把消费位移保存到你自己的地方  
  mysql/zk/redis/ -> 提交到自己所涉及的存储；初始化时也需要自己去从自定义存储中查询到消费位移</b></font>





## 6.7 其它重要参数

- <font size=3><b>fetch.min.bytes=1B    （一次拉取的最小字节数）</b></font>
- <font size=3><b>fetch.max.bytes=50M   （一次拉取的最大数据量）</b></font>
- <font size=3><b>fetch.max.wait.ms=500ms    (拉取时的最大等待时长</b></font>)
- <font size=3><b>max.partition.fetch.bytes=1MB    （每个分区一次拉取的最大数据量）</b></font>
- <font size=3><b>max.poll.records=500    （一次拉取的最大条数）</b></font>
- <font size=3><b>connections.max.idle.ms=540000ms    （网络连接的最大闲置时长）</b></font>
- <font size=3><b>request.timeout.ms=30000ms    （一次请求等待响应的最大超时时长，consumer等待请求响应的最长时间）</b></font>
- <font size=3><b>metadata.max.age.ms=300000    （元数据在限定时间内没有进行更新，则会被强制更新）</b></font>
- <font size=3><b>reconnect.backoff.ms=50ms    （尝试重新连接指定主机之前的退避时间）</b></font>
- <font size=3><b>retry.backoff.ms=100ms    （尝试重新拉取数据的重试间隔）
</b></font>

# 7. topic管理 API示例

<font size=3><b>如果希望将管理类的功能集成到公司内部的系统中，打造集管理、监控、运营、告警为一体的生态平台，那么就需要以程序调用API方式去实现。   
工具类KafkaAdminClient可以用来管理broker、配置和ACL(Access Control List)，管理topic。
</b></font>

- <font size=3><b>创建主题：CreateTopicsResult createTopics(Collection\<NewTopic\> newTopics)</b></font>
- <font size=3><b>删除主题：DeleteTopicsResult deleteTopics(Collection\<String\> topics)</b></font>
- <font size=3><b>列出所有可用的主题：ListTopicsResult listTopics()</b></font>
- <font size=3><b>查看主题的信息：DescribeTopicsResult describeTopics(Collection\<String\> topicNames)</b></font>
- <font size=3><b>查询配置信息：DescribeConfigsResult describeConfigs(Collection<\ConfigResource\>
 resources)</b></font>
- <font size=3><b>修改配置信息：AlterConfigsResult alterConfigs(Map\<ConfigResource,Config\> configs)</b></font>
- <font size=3><b>增加分区：CreatePartitionsResult createPartitions(Map\<String,NewPartitions\> newPartitions)
</b></font>

<font size=3><b>构造一个KafkaAdminClient
</b></font>

```java
AdminClient adminClient = KafkaAdminClient.create(props);
```

## 7.1 列出主题

```java
ListTopicsResult listTopicsResult = adminClient.listTopics();
Set<String> topics = listTopicsResult.names().get();
System.out.printIn(topics);
```

## 7.2 查看主题信息

```java
DescribeTopicsResult describeTopicsResult = adminClient.describeTopics(Arrays.asList ("tpc_4", "tpc_3"));
Map<String, TopicDescription> res = describeTopicsResult.all().get();
Set<String> ksets = res.keySet();
for (String k : ksets) {
    System.out.println(res.get(k));
}
```


## 7.3 创建主题


```java
//参数配置
Properties props = new Properties();
props.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "doit01:9092,doit02:9092,doit03:
9092")；
props.put(AdminClientConfig.REQUEST_TIMEOUT_MS_CONFIG, 3000);
//创建admin client对象
AdminClient adminClient = KafkaAdminClient.create(props);
//由服务端controller自行分配分区及副本所在broker
NewTopic tpc_3 = new NewTopic("tpc_3", 2, (short) 1);
//手动指定分区及副本的broker分配
HashMap<Integer, List<Integer>> replicaAssignments = new HashMap<>();
//分区0，分配到broker0,broker1
replicaAssignments.put(0, Arrays.asList(0, 1));
//分区1，分配到broker0,broker2
replicaAssignments.put(1, Arrays.asList(0, 1));

NewTopic tpc_4 = new NewTopic("tpc_4", replicaAssignments);
CreateTopicsResult result = adminClient.createTopics(Arrays.asList(tpc_3, tpc_4));

//从future中等待服务端返回
try{
    result.all().get()；
} catch (Exception e){
    e.printStackTrace();
}
adminClient.close();
```

## 7.4 删除主题


```java
DeleteTopicsResult deleteTopicsResult = adminClient.deleteTopics(Arrays.asList("tpc_1", 
"tpc_1")):
Map<String, KafkaFuture<Void>> values = deleteTopicsResult.values();
System.out.printIn(values);
```

## 7.5 其它管理

<font size=3><b>除了进行topic管理外，KafkaAdminClient也可以进行诸如动态参数管理，分区管理等各类管理操作；
</b></font>
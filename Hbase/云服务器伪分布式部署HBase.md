## 云服务器伪分布式部署HBase


## 1.解压tar包

这里我解压到/home/software/目录下
```shell
tar -zxvf hbase-2.3.6-bin.tar.gz  -C /home/software/
```

## 2.配置conf

### 2.1 hbase-env.sh

进入conf目录，在hbase-env.sh文件中添加如下，这里我不用HBase自带的Zookeeper，下面HBASE_MANAGES_ZK设置为fales，使用独立配置的Zookeeper。
```shell
export JAVA_HOME=/home/software/jdk1.8.0_202
export HBASE_MANAGES_ZK=false
```

### 2.2 hbase-env.sh


在hbase-site.xml文件中添加如下
```xml
<property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>

  <property>
        <name>hbase.rootdir</name>
        <value>hdfs://192.168.0.219:9000/hbase</value>
  </property>

  <property>
        <name>hbase.zookeeper.quorum</name>
        <value>192.168.0.219</value>
  </property>

  <!--配置zk本地数据存放目录 -->
  <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>/home/software/hbase-2.3.6/zookeeper</value>
  </property>
```

### 2.2 regionservers

在regionservers文件中添加服务器ip地址

## 3.配置环境变量

```shell
export HBASE_HOME=/home/software/hbase-2.3.6
export PATH=$PATH:/$HBASE_HOME/bin
```

## 3.启动

注意，在启动HBase之前，先启动Zookeeper，然后启动HBase
```shell
start-hbase.sh
```

jps进程出现HMaster，HRegionServer则说明启动成功，在浏览器输入IP地址和端口号16010，出现HMaster的Web界面。
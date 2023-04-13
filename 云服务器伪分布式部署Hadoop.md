## 云服务器伪分布式部署Hadoop


## 1.hadoop安装包下载 [下载地址](https://downloads.apache.org/hadoop/common/)
下载后缀名为.tar.gz的文件

## 2.上传安装包到服务器并解压

这里我解压到/home/software目录下

```sh
tar -zxvf hadoop-3.2.2.tar.gz  -C /home/software/
```

## 3.配置ssh免密登录

```shell
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa

cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

chmod 0600 ~/.ssh/authorized_keys
```


## 4.添加环境变量

修改/etc/profile
```shell
vim /etc/profile
```

添加如下

```shell
export HADOOP_HOME=/home/software/hadoop-3.2.2
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```

使环境变量立即生效
```shell
source /etc/profile
```
## 5.配置hadoop
进入hadoop安装目录下的etc/hadoop文件夹
```shell
cd /home/software/hadoop-3.2.2/etc/hadoop
```

### 5.1 配置hadoop-env.sh

```shell
vim hadoop-env.sh
```
添加如下

```shell
export JAVA_HOME=/home/software/jdk1.8.0_202

export HDFS_DATANODE_USER=root
export HDFS_NAMENODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
```

### 5.2 配置core-site.xml

```shell
vim core-site.xml
```
```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://192.168.0.219:9000</value>
    </property>

<!--配置HDFS数据块和元数据保存的目录，一定要修改-->
    <property>
         <name>hadoop.tmp.dir</name>
         <value>/home/software/hadoop-3.2.2/tmp</value>
   </property>
</configuration>
```
注意，要在hadoop目录下创建tmp目录


### 5.3 配置hdfs-site.xml
```shell
vim hdfs-site.xml
```
```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>

    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/home/software/hadoop-3.2.2/tmp/nameNodeDir</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/home/software/hadoop-3.2.2/tmp/dataNodeDir</value>
    </property>
</configuration>
```
注意，伪分布式虽然只需要配置 fs.defaultFS 和 dfs.replication 就可以运行（官方教程如此），不过若没有配置 hadoop.tmp.dir 参数，则默认使用的临时目录为 /tmp/hadoo-hadoop，而这个目录在重启时有可能被系统清理掉，导致必须重新执行 format 才行。所以我们进行了设置，同时也指定 dfs.namenode.name.dir 和 dfs.datanode.data.dir，否则在接下来的步骤中可能会出错。

### 5.4 配置mapred-site.xml
```shell
vim mapred-site.xml
```

```xml
<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>

	<!--历史服务器端地址-->
        <property>
                <name>mapreduce.jobhistory.address</name>
                <value>192.168.0.219:10020</value>
        </property>
        <!-- 历史服务器web端地址 -->
        <property>
                <name>mapreduce.jobhistory.webapp.address</name>
                <value>192.168.0.219:19888</value>
        </property>
</configuration>
```


### 5.5 配置yarn-site.xml
```shell
vim yarn-site.xml
```

yarn默认端口是8088，但是这里必须要修改yarn端口，原因是因为像yarn、redis这种8088和6379端口是近年来挖矿病毒的热门踩点端口，具体的可参考这篇文章[https://segmentfault.com/a/1190000015264170](https://segmentfault.com/a/1190000015264170)

```xml
<configuration>

<!-- Site specific YARN configuration properties -->

        <!--配置Yarn的节点-->
        <property>
           <name>yarn.resourcemanager.hostname</name>
           <value>192.168.0.219</value>
        </property>
        <!--NodeManager执行MR任务的方式是Shuffle洗牌-->
        <property>
           <name>yarn.nodemanager.aux-services</name>
           <value>mapreduce_shuffle</value>
        </property>
	<!--修改yarn端口为7776-->
        <property>
                <name>yarn.resourcemanager.webapp.address</name>
                <value>192.168.0.219:7776</value>
        </property>
</configuration>
```

## 2022/6/21 更新：以下步骤不需要了，直接跳过

~~### 5.5 sbin目录修改4个文件
在Hadoop安装目录下找到sbin文件夹
在里面修改四个文件
对于start-dfs.sh和stop-dfs.sh文件，在文件开头添加下列参数：~~
```shell
#!/usr/bin/env bash
HDFS_DATANODE_USER=root
HDFS_DATANODE_SECURE_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root
```

~~对于start-yarn.sh和stop-yarn.sh文件，在文件开头添加下列参数：~~
```shell
#!/usr/bin/env bash
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
```


## 6.执行 NameNode 的格式化
进入hadoop目录下的bin目录
```shell
cd /home/software/hadoop-3.2.2/bin/
```
格式化NameNode
```shell
hdfs namenode -format
```
成功的话，会看到 “Storage directory XXX/XXX/XXX has been successfully formatted” 的提示。


## 6.启动

### 6.1启动hdfs
进入hadoop目录下的bin目录，启动hdfs
```shell
./start-all.sh
```
查看java进程
```shell
jps
```
如有以下5个进程，说明hadoop伪分布部署成功
NameNode
SecondaryNameNode
ResourceManager
NodeManager
DataNode

访问服务器9870端口，出现hadoop的Web界面。
访问服务器7776端口，出现yarn界面。


### 6.2启动历史服务器

```shell
./mapred --daemon start historyserver
```
出现JobHistoryServer进程，访问服务器19888端口，出现hadoop的JobHistory界面。
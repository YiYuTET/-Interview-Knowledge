# 云服务器伪分布式部署Spark

## 1.下载Spark [下载地址](https://spark.apache.org/downloads.html)

安装spark前需要安装scala （[scala下载地址](https://www.scala-lang.org/download/)）

## 2.上传至服务器并解压

这里我解压到/home/software/目录下
```shell
tar -zxvf spark-3.1.2-bin-hadoop3.2.tgz  -C /home/software/
```

## 3.配置环境变量

```shell
vim /etc/profile
```

添加如下
```shell
#这里没有添加sbin目录，因为会和hadoop的sbin目录冲突
export SPARK_HOME=/home/software/spark-3.1.2-bin-hadoop3.2
export PATH=$PATH:$SPARK_HOME/bin
```


## 4.配置conf

### 4.1 配置spark-env.sh

进入conf目录
```shell
cd /home/software/spark-3.1.2-bin-hadoop3.2/conf/
```

复制spark-env.sh.template，重命名为spark-env.sh
```shell
cp spark-env.sh.template spark-env.sh
```


添加如下
```sh
export JAVA_HOME=/home/software/jdk1.8.0_202
export HADOOP_HOME=/home/software/hadoop-3.2.2
export HADOOP_CONF_DIR=/home/software/hadoop-3.2.2/etc/hadoop
export YARN_CONF_DIR=/home/software/hadoop-3.2.2/etc/hadoop

export SPARK_DIST_CLASSPATH=$(/home/software/hadoop-3.2.2/bin/hadoop classpath)

export SCALA_HOME=/home/software/scala-2.12.4
export SPARK_HOME=/home/software/spark-3.1.2-bin-hadoop3.2

export SPARK_MASTER_IP=192.168.0.219
export SPARK_MASTER_PORT=7077

#spark master节点的网页端口（默认是8080）可自行设置
#export SPARK_MASTER_WEBUI_PORT=8080
export SPARK_WORKER_CORES=2
export SPARK_WORKER_INSTANCES=1
export SPARK_WORKER_MEMORY=2G

#spark worker节点的网页端口（默认是8081）可自行设置
#export SPARK_WORKER_WEBUI_PORT=8081
export SPARK_EXECUTOR_CORES=1
export SPARK_EXECUTOR_MEMORY=1G
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:$HADOOP_HOME/lib/native
```

内容说明：

| 变量名                  | 说明                                       |
| ----------------------- | :----------------------------------------- |
| JAVA_HOME               | jdk的安装目录                              |
| HADOOP_HOME             | hadoop的安装目录                           |
| HADOOP_CONF_DIR         | hadoop的配置文件存放目录                   |
| SCALA_HOME              | scala的安装目录                            |
| SPARK_HOME              | spark的安装目录                            |
| SPARK_MASTER_IP         | spark主节点绑定的ip地址                    |
| SPARK_MASTER_PORT       | spark主节点绑定的端口号                    |
| SPARK_MASTER_WEBUI_PORT | spark master节点的网页端口（默认是8088）   |
| SPARK_WORKER_CORES      | worker使用的cpu核心数                      |
| SPARK_WORKER_INSTANCES  | 最多能够同时启动的EXECUTOR的实例个数       |
| SPARK_WORKER_MEMORY     | worker分配的内存数量                       |
| SPARK_WORKER_WEBUI_PORT | worker的网页查看绑定的端口号（默认是8081） |
| SPARK_EXECUTOR_CORES    | 每个executor分配的cpu核心数                |
| SPARK_EXECUTOR_MEMORY   | 每个executor分配的内存数                   |
| LD_LIBRARY_PATH         | 指定查找共享库                             |


### 4.2 配置workers
复制workers.template，重命名为workers
```shell
cp workers.template workers
```
添加ip地址
```shell
192.168.0.219
```


### 4.3 配置spark-defaults.conf
复制spark-defaults.conf.template，重命名为spark-defaults.conf
```shell
cp spark-defaults.conf.template spark-defaults.conf
```

```shell
spark.master                     spark://192.168.0.219:7077
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs://192.168.0.219:9000/sparkLogs
spark.history.fs.logDirectory    hdfs://192.168.0.219:9000/sparkLogs
```

内容说明：

| 变量名                        | 说明                                          |
| :---------------------------- | --------------------------------------------- |
| spark.master                  | spark主节点所在机器及端口，默认写法是spark:// |
| spark.eventLog.enabled        | 是否打开任务日志功能，默认为false             |
| spark.eventLog.dir            | 任务日志默认存放位置，配置为一个HDFS路径即可  |
| spark.history.fs.logDirectory | 存放历史应用日志文件的目录                    |

## 5.启动
进入spark的sbin目录下
```shell
cd /home/software/spark-3.1.2-bin-hadoop3.2/sbin/
```
启动spark
```shell
./start-all.sh
```

因为这里没有配置spark/sbin目录的环境变量 所以需要cd到spark的sbin目录下再进行启动（没配置此目录的环境变量是因为spark的启动文件 start-all.sh与hadoop的启动文件名重名，配了会发生冲突，解决办法可以将两个文件中的其中一个重命名即可，这里读者就没有进行相关的操作了，是直接全路径指定执行启动的）

查看java进程
```shell
jps
```
如有Master，Worker两个进程则说明伪分布式部署成功！
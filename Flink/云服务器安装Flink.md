# 云服务器安装Flink

## 1.Flink[下载地址](https://flink.apache.org/downloads.html)

## 2.解压

这里我解压到/home/software目录下
```shell
tar -zxvf flink-1.13.2-bin-scala_2.11.tgz  -C /home/software/
```

## 3.配置环境变量

```shell
export FLINK_HOME=/home/software/flink-1.13.2
export PATH=$PATH:/$FLINK_HOME/bin
export HADOOP_CONF_DIR=/home/software/hadoop-3.2.2/etc/hadoop
```


## 4.配置conf文件
修改conf目录下的flink-conf.yaml文件

```shell
vim flink-conf.yaml
```

```yaml
jobmanager.rpc.address: 192.168.0.219

state.backend: filesystem

state.backend.fs.checkpointdir: hdfs://192.168.0.219:9000/flink-checkpoints
state.savepoints.dir: hdfs://192.168.0.219:9000/flink-savepoints

high-availability: zookeeper
high-availability.zookeeper.quorum: 192.168.0.219:2181
high-availability.storageDir: hdfs://192.168.0.219:9000/flink/ha/
high-availability.zookeeper.client.acl: open
```

修改masters的IP地址

```shell
114.116.24.98:8082
```
修改workers的IP地址
```shell
114.116.24.98
```

修改conf下zoo.cfg配置
```shell
dataDir=/home/software/flink-1.13.2/tmp/zookeeper
server.1=localhost:2887:3887
```
## 5.启动
进入bin目录下，启动Flink
```shell
./start-cluster.sh
```
jps查看进程出现TaskManagerRunner和StandaloneSessionClusterEntrypoint

在浏览器输入http://114.116.24.98:8082可进入Flink的Web页面
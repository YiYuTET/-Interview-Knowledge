# Hue集成HBase

## 1.修改hbase-site.xml

增加如下配置
```xml
  <!--指定regionserver thrift端口号 默认9090 -->
  <property>
    <name>hbase.regionserver.thrift.port</name>
    <value>1124</value>
  </property>
```

## 2.启动HBase，thrift服务

```shell
start-hbase.sh

hbase-daemon.sh start thrift -p 1124
```


## 3.修改hue.ini

```ini
[hbase]

hbase_clusters=(HBase|192.168.0.219:1124)

hbase_conf_dir=/home/software/hbase-2.3.6/conf
```


## 4.启动Hue

进入hue根目录
```sh
cd /home/software/hue-4.7.1/hue
```
启动hue并挂在到后台

```shell
nohup ./build/env/bin/supervisor &
```
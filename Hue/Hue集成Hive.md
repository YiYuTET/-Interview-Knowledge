# Hue集成Hive

## 1.时间同步

安装ntp
```shell
yum -y install ntp
```

与上海电信服务器同步时间
```shell
ntpdate api.ntp.bz
```


## 2.修改hive-site.xml

进入hive的conf目录下
```shell
cd /home/software/apache-hive-3.1.2-bin/conf
```

增加hive-site.xml配置
```xml
    <!-- hiveserver2运行绑定host -->
    <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>192.168.0.219</value>
    </property>

    <!-- hiveserver2运行绑定端口 -->
    <property>
        <name>hive.server2.thrift.port</name>
        <value>11240</value>
    </property>

    <!-- 远程模式部署metastore服务地址 -->
    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://192.168.0.219:9083</value>
    </property>
```


## 2.启动Hadoop

```shell
start-all.sh

#启动任务历史服务器
mapred --daemon start historyserver
```

## 3.修改hue.ini

进入hue的conf目录
```shell
cd /home/software/hue-4.7.1/hue/desktop/conf
```

<font color=#dd0000>修改hue.ini</font>
```ini
[beeswax]

hive_server_host=192.168.0.219
#hiveserver2的thrift端口号
hive_server_port=11240
hive_conf_dir=/home/software/apache-hive-3.1.2-bin/conf

server_conn_timeout=120

auth_username=root
auth_password=123456



[metastore]
#允许使用hive创建数据库表等操作
enable_new_create_table=true
```

## 4.启动hiveserver2和metastore服务

<font color=#dd0000>启动metastore</font>

```shell
#
nohup hive --service metastore &
```

<font color=#dd0000>启动hiveserver2</font>
```shell
nohup hive --service hiveserver2 &

#或者
nohup hiveserver2 &
```

## 5.启动hue

进入hue根目录
```shell
cd /home/software/hue-4.7.1/hue
```

启动hue并挂在到后台
```shell
nohup ./build/env/bin/supervisor &
```
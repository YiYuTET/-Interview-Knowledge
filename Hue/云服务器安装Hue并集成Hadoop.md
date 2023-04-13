## 云服务器安装Hue

## 1.源码下载

Hue官网并没有提供二进制安装包，我们需要自行编译。
源码文件下载地址[https://github.com/cloudera/hue/releases](https://github.com/cloudera/hue/releases)

## 1.上传解压
这里我解压到/home/software/sourceSoftware/目录下
```shell
tar -zxvf hue-release-4.7.1.tar.gz  -C /home/software/sourceSoftware/
```

## 2.安装依赖包

在编译之前必须安装各种依赖软件。
```shell
yum -y install ant asciidoc cyrus-sasl-devel cyrus-sasl-gssapi cyrus-sasl-plain gcc gcc-c++ krb5-devel libffi-devel libxml2-devel libxslt-devel make mysql mysql-devel openldap-devel python-devel sqlite-devel gmp-devel rsync openssl-devel
```


## 3.源码编译
进入源码文件解压根目录
```shell
cd /home/software/sourceSoftware/hue-release-4.7.1
```

执行以下命令进行安装
```shell
PREFIX=/home/software/hue-4.7.1 make install
```

编译完成后在/home/software/hue-4.7.1目录下就会有hue软件包


## 4.配置环境变量

```shell
vim /etc/profile
```

```shell
export HUE_HOME=/home/software/hue-4.7.1/hue
export PATH=$PATH:/$HUE_HOME/build/env/bin
```

```shell
source /etc/profile
```


## 5.修改配置

在 core-site.xml 中增加配置

```xml
   <property>
        <name>hadoop.proxyuser.root.hosts</name>
        <value>*</value>
        </property>
   <property>
        <name>hadoop.proxyuser.root.groups</name>
        <value>*</value>
   </property>


   <!-- HUE -->
   <!-- #设置Hadoop集群的代理用户 -->
   <property>
        <name>hadoop.proxyuser.hue.hosts</name>
        <value>*</value>
   </property>
   <!-- #设置Hadoop集群的代理用户组 -->
   <property>
        <name>hadoop.proxyuser.hue.groups</name>
        <value>*</value>
   </property>
```

在 hdfs-site.xml 中增加配置

```xml
    <property>
         <name>dfs.webhdfs.enabled</name>
         <value>true</value>
    </property>
    <property>
        <name>dfs.permissions.enabled</name>
        <value>false</value>
    </property>

<!-- datanode 通信是否使用域名,默认为false，改为true -->
    <property>
        <name>dfs.client.use.datanode.hostname</name>
        <value>true</value>
        <description>Whether datanodes should use datanode hostnames when
        connecting to other datanodes for data transfer.
        </description>
    </property>
```

在yarn-site.xml中增加配置
```xml
<!--打开HDFS上日志记录功能-->
    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>

<!--在HDFS上聚合的日志最长保留多少秒。3天-->
    <property>
         <name>yarn.log-aggregation.retain-seconds</name>
         <value>259200</value>
    </property>
```

在mapred-site.xml中增加配置，否则运行mapreduce任务会报错。
```shell
	<property>
                <name>yarn.app.mapreduce.am.env</name>
                <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
        </property>
        <property>
                <name>mapreduce.map.env</name>
                <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
        </property>
        <property>
                <name>mapreduce.reduce.env</name>
                <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
        </property>
```

增加 httpfs-site.xml 文件，加入配置

```xml
<!-- HUE -->        
<property>                
	<name>httpfs.proxyuser.hue.hosts</name>                
	<value>*</value>        
</property>        
<property>                
	<name>httpfs.proxyuser.hue.groups</name>                
	<value>*</value>        
</property>
```

## 6.配置hue.ini

进入hue的desktop/conf目录，里面有一个pseudo-distributed.ini.tmpl文件，复制一份改名为hue.ini
```shell
/home/software/hue-4.7.1/hue/desktop/conf
```

```shell
cp pseudo-distributed.ini.tmpl  hue.ini
```

修改hue.ini

```ini
[desktop]

secret_key=jFE93j;2[290-eiw.KEiwN2s3['d;/.q[eIW^y#e=+Iei*@Mn<qW5o

http_host=192.168.0.219
http_port=8000

time_zone=Asia/Shanghai

server_user=root
server_group=root

default_user=root
default_hdfs_superuser=root


[[database]]

    engine=mysql
    host=192.168.0.219
    port=3306
    user=root
    password=123456
    name=hue


[hadoop]

[[hdfs_clusters]]
[[[default]]]

fs_defaultfs=hdfs://192.168.0.219:9000

webhdfs_url=http://192.168.0.219:9870/webhdfs/v1

hadoop_conf_dir=/home/software/hadoop-3.2.2/etc/hadoop
hadoop_hdfs_home=/home/software/hadoop-3.2.2
hadoop_bin=/home/software/hadoop-3.2.2/bin


[[yarn_clusters]]
[[[default]]]

resourcemanager_host=192.168.0.219

resourcemanager_port=8032

resourcemanager_api_url=http://192.168.0.219:7776
proxy_api_url=http://192.168.0.219:7776
history_server_api_url=http://192.168.0.219:19888
```

在mysql中创建hue数据库
```sql
create database hue default character set utf8 default collate utf8_general_ci;
```

去hue/build/env/bin目录下，执行以下命令，在mysql的hue数据库里看到对应的hue元数据。
```shell
#新建数据库并初始化
cd hue/build/env/bin
./hue syncdb
./hue migrate
```


## 7.启动hue

进入hue根目录

```shell
cd /home/software/hue-4.7.1/hue
```

这里必须登录zyw用户
```shell
su zyw
```

启动hue并挂在到后台
```shell
nohup ./build/env/bin/supervisor &
```

启动之后，在Web浏览器访问8000端口即可进入hue界面。
**登录用户：root
密码：123456**
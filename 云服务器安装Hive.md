# 云服务器安装Hive

## 1.下载Hive([下载地址](https://mirrors.tuna.tsinghua.edu.cn/apache/hive/))

## 2.上传至服务器并解压

这里我解压到/home/software/目录下
```shell
tar -zxvf apache-hive-3.1.2-bin.tar.gz  -C /home/software/
```

## 3.添加MySQL驱动包，拷贝到Hive的安装目录的依赖库目录下

```shell
cp mysql-connector-java-5.1.46-bin.jar /home/software/apache-hive-3.1.2-bin/lib/
```


## 4.添加环境变量

```shell
vim /etc/profile
```

添加如下
```shell
export HIVE_HOME=/home/software/apache-hive-3.1.2-bin
export PATH=$PATH:/$HIVE_HOME/bin
```
使环境变量立即生效
```shell
source /etc/profile
```

## 4.修改配置文件
### 4.1 配置hive-env.sh文件
进入hive下的conf目录
```shell
cd /home/software/apache-hive-3.1.2-bin/conf/
```

拷贝hive-env.sh.template文件，并命名为hive-env.sh
```shell
cp hive-env.sh.template  hive-env.sh
```

添加如下
```shell
export HADOOP_HOME=/home/software/hadoop-3.2.2

export HIVE_CONF_DIR=/home/software/apache-hive-3.1.2-bin/conf

export HIVE_AUX_JARS_PATH=/home/software/apache-hive-3.1.2-bin/lib
```
第一个为Hadoop目录，第二个为Hive配置目录，最后一个为驱动jar包路径


### 4.2 配置hive-site.xml文件


新建hive-site.xml文件
```shell
vim hive-site.xml
```
添加如下
```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <!-- 本地模式 -->
       <property>
           <name>hive.metastore.local</name>
           <value>true</value>
       </property>
    <!-- jdbc 连接的 URL -->
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://192.168.0.219:3306/hive?createDatabaseIfNotExist=true&amp;characterEncoding=UTF-8&amp;useSSL=false</value>
    </property>

     <!-- jdbc 连接的 Driver-->
        <!--新版本8.0版本的驱动为com.mysql.cj.jdbc.Driver-->
        <!--旧版本5.x版本的驱动为com.mysql.jdbc.Driver-->
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>

    <!-- jdbc 连接的 username(MySQL用户名)-->
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>

    <!-- jdbc 连接的 password(MySQL密码) -->
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
    </property>

     <!-- Hive 元数据存储版本的验证(Hive元数据默认是存储在Derby中，正常开启时它会去校验Derby，现在要使用MySQL存储元数据，
               就需要把这个关闭即可，如果开启，MySQL和Derby会导致Hive启动不起来的) -->
    <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
    </property>

    <!-- Hive  默认在 HDFS 的工作目录(可以不配置，因为默认就是/user/hive/warehouse，如果不使用默认的位置，可以进行手动修改) -->
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
    </property>

    <!-- hiveserver2运行绑定host -->
    <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>192.168.0.219</value>
    </property>

    <!-- 自定义hiveserver2运行绑定端口，默认10000 -->
    <property>
        <name>hive.server2.thrift.port</name>
        <value>11240</value>
    </property>

    <!-- 远程模式部署metastore 服务地址 -->
    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://192.168.0.219:9083</value>
    </property>
 
    <!-- 关闭元数据存储授权  -->
    <property>
        <name>hive.metastore.event.db.notification.api.auth</name>
        <value>false</value>
    </property>
</configuration>
```

在HDFS中创建hive工作目录
```shell
hadoop fs -mkdir -p /user/hive/warehouse
```


## 5.配置log日志文件

拷贝并重命名hive-log4j2.properties.template为 hive-log4j2.properties文件
```shell
cp hive-log4j2.properties.template  hive-log4j2.properties
```
修改内容 property.hive.log.dir 这个属性
```shell
property.hive.log.dir = /home/software/apache-hive-3.1.2-bin/temp
```
并在hive目录下创建temp文件夹
```shell
mkdir /home/software/apache-hive-3.1.2-bin/temp
```

## 6.初始化元数据库

进入hive安装目录下的bin目录
```shell
cd /home/software/apache-hive-3.1.2-bin/bin
```

初始化
```shell
./schematool -dbType mysql -initSchema
```
可能出现问题，[解决方案](https://blog.csdn.net/m0_59705760/article/details/125116629)

出现Initialization script completed
schemaTool completed则说明初始化成功

## 7.启动Hive

### 7.1 本地模式启动

本地模式特点是：**需要安装MySQL来存储元数据，但是不需要启动metastore服务**
直接使用hive命令在本地启动客户端

```shell
hive
```

### 7.2 远程模式启动

远程模式特点是：**需要安装MySQL来存储元数据，需要单独启动metastore服务**

*nohup 英文全称 no hang up（不挂起），用于在系统后台不挂断地运行命令，退出终端不会影响程序的运行。
nohup 命令，在默认情况下（非重定向时），会输出一个名叫 nohup.out 的文件到当前目录下，如果当前目录的 nohup.out 文件不可写，输出重定向到 $HOME/nohup.out 文件中。
参数说明：&：让命令在后台执行，终端退出后命令仍旧执行。*

这里使用nohup命令将将HiveServer2远程连接进程在后台运行
```shell
nohup hive --service hiveserver2 &
```

HiveServer2通过metastore服务读写元数据，所以远程模式下，启动HiveHiveServer2之前必须启动metastore服务
先开启metastore服务
```shell
nohup hive --service metastore &
```

再启动HiveServer2
```shell
nohup hive --service hiveserver2 &
```

## 8.Hive客户端

第一代客户端（不推荐使用）：$HIVE_HOME/bin/hive，是一个shellUtil，主要功能：一是可以用于交互或批处理模式运行Hive查询；二是用于Hive相关服务的启动，比如metastore服务。

第二代客户端（推荐使用）：$HIVE_HOME/bin/beeline，是一个JDBC客户端，是官方强烈推荐使用的Hive命令行工具，和第一代客户端相比，性能加强安全性提高。
**在远程模式下，beeline需要通过Thrift连接到单独的HiveServer2服务上，所以在使用前需要启动metastore服务和HiveServer2服务。**

```shell
!connect jdbc:hive2://192.168.0.219:11240

Connecting to jdbc:hive2://192.168.0.219:11240
Enter username for jdbc:hive2://192.168.0.219:11240: root
Enter password for jdbc:hive2://192.168.0.219:11240: 123456
```

或者一步到位：

```shell
beeline -u jdbc:hive2://192.168.0.219:11240 -n root
```

设置mapreduce本地运行

```sh
set mapreduce.framework.name=local;
```
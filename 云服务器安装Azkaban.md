# 云服务器安装Azkaban

## 1.源码编译

Azkaban官方并没有提供二进制安装包，需要我们自行编译。
[源码文件下载](https://azkaban.github.io/)


### 1.1编译环境

编译之前需要提前安装好Maven，Ant，Node等软件，还有git和gcc-c++环境。

```shell
yum  install -y git
```
```shell
yum  install -y gcc-c++
```

### 1.1下载源码解压

这里我解压到/home/software/目录下
```shell
tar -zxvf azkaban-4.0.0.tar.gz -C /home/software/
```

### 1.2源码文件编辑

在编译之前对文件进行修改，避免编译过程出现问题。

### 1.2.1修改build.gradle文件

```shell
vim build.gradle
```

```gradle
repositories {
    mavenCentral()
    maven {
      url 'https://maven.aliyun.com/repository/central'
    }
    maven {
      url 'https://maven.aliyun.com/repository/gradle-plugin'
    }
    maven {
      url 'https://maven.aliyun.com/repository/public'
    }
    maven {
      url 'https://maven.aliyun.com/repository/google'
    }
    maven {
      url 'https://maven.aliyun.com/nexus/content/groups/public/'
    }
    maven {
      url 'https://plugins.gradle.org/m2/'
    }
  }
```
具体参考[https://blog.csdn.net/chenxi5404/article/details/120512109](https://blog.csdn.net/chenxi5404/article/details/120512109)


```gradle
allprojects {
  apply plugin: 'jacoco'

  repositories {
    mavenCentral()
    mavenLocal()
//  need this for rest.li/pegasus 28.* artifacts until they are in Maven Central:
    maven {
      url 'https://linkedin.jfrog.io/artifactory/open-source/'
    }
  }
}
```
具体参考[https://blog.csdn.net/NKDark0214/article/details/122601181](https://blog.csdn.net/NKDark0214/article/details/122601181)


### 1.2.2修改gradle-wrapper.properties文件
Azkaban使用的是gradle进行构建的，我这里直接下载一个gradle安装包，使用的版本为4.6的。
把下载好的安装包上传到服务器上并移动到Azkaban的源码目录中。

```shell
cp gradle-4.6-bin.zip /home/software/azkaban-4.0.0/gradle/wrapper
```
修改这个目录的gradle-wrapper.properties配置文件
```shell
vim gradle-wrapper.properties
```

```properties
distributionUrl=gradle-4.6-bin.zip
```

### 1.3编译

```shell
./gradlew build installDist -x test
```

出现BUILD SUCCESSFUL表示编译成功。
然后在
azkaban-exec-server/build/distributions/
azkaban-web-server/build/distributions/
azkaban-db/build/distributions/
这些目录下会有相关安装包。

## 2.解压安装包

创建azkaban安装目录

```shell
mkdir /home/software/azkaban
```

```shell
tar -zxvf azkaban-db-0.1.0-SNAPSHOT.tar.gz  -C /home/software/azkaban
tar -zxvf azkaban-exec-server-0.1.0-SNAPSHOT.tar.gz  -C /home/software/azkaban
tar -zxvf azkaban-web-server-0.1.0-SNAPSHOT.tar.gz  -C /home/software/azkaban
```

## 3.MySQL初始化

```shell
cd /home/software/azkaban/azkaban-db-0.1.0-SNAPSHOT
```

创建azkaban数据库，并加载初始化sql脚本。

```sql
create database azkaban;

use azkaban;

source /home/software/azkaban/azkaban-db-0.1.0-SNAPSHOT/create-all-sql-0.1.0-SNAPSHOT.sql
```


## 4.web-server服务器配置

```shell
cd /home/software/azkaban
```

### 4.1生成ssl证书
```shell
keytool -keystore keystore -alias jetty -genkey -keyalg RSA
```
运行此命令后，会提示输入当前生成keystore的密码和相关信息，输入的密码请记住（所以密码均为123456）。

然后将证书复制到web-server服务器根目录下

```shell
cp keystore azkaban-web-server-0.1.0-SNAPSHOT/
```

### 4.2配置azkaban.properties

```shell
cd /home/software/azkaban/azkaban-web-server-0.1.0-SNAPSHOT/conf
```

修改如下
```properties
default.timezone.id=Asia/Shanghai

jetty.use.ssl=true
jetty.port=8443

executor.host=192.168.0.219
executor.port=12321

jetty.keystore=keystore
jetty.password=123456
jetty.keypassword=123456
jetty.truststore=keystore
jetty.trustpassword=123456

mysql.host=192.168.0.219
mysql.database=azkaban
mysql.user=root
mysql.password=123456

azkaban.use.multiple.executors=true
#注释
#azkaban.executorselector.filters=StaticRemainingFlowSize,MinimumFreeMemory,CpuStatus
```

### 4.3配置commonprivate.properties

新建文件夹
```shell
mkdir -p plugins/jobtypes
```

新建commonprivate.properties文件
```shell
vim commonprivate.properties
```

添加如下
```properties
azkaban.native.lib=false
execute.as.user=false
memCheck.enabled=false
```

## 4.exec-server服务器配置

```shell
cp /home/software/azkaban/azkaban-exec-server-0.1.0-SNAPSHOT/conf
```

### 4.1配置azkaban.properties

```shell
vim azkaban.properties
```

修改如下
```properties
default.timezone.id=Asia/Shanghai

azkaban.webserver.url=https://192.168.0.219:8443

mysql.host=192.168.0.219
mysql.database=azkaban
mysql.user=root
mysql.password=123456

executor.port=12321
```


## 5.启动
先启动exec-server
再启动web-server
```shell
bin/start-exec.sh
```
```shell
bin/start-web.sh
```
**注意，必须在安装包根目录下执行**
启动web-server后进程失败，可以通过安装包下对应启动日志进行排查

访问[https://114.116.24.98:8443](https://114.116.24.98:8443)
进入AzkabanWeb页面
**Username：azkaban
password：azkaban**

## 6.exec-server激活问题

**每次关闭exec-server后都要重新激活execute**

进入exec-server根目录
```shell
cd /home/software/azkaban/azkaban-exec-server-0.1.0-SNAPSHOT
```

```shell
curl -G "192.168.0.219:$(<./executor.port)/executor?action=activate" && echo
```
然后再重新启动web-server
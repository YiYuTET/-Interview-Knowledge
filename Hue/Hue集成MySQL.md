# Hue集成MySQL


## 1.添加MySQL方言

进入hue根目录，添加到Hue Python虚拟环境中
```shell
./build/env/bin/pip install mysqlclient
```

## 2.修改hue.ini
需要把mysql的注释去掉，大概在1821行。
```ini
[notebook]

[[interpreters]]

[[[mysql]]]
name = MySQL
interface=sqlalchemy
options='{"url": "mysql://root:123456@192.168.0.219:3306/"}'

[[[hive]]]
name=Hive
interface=hiveserver2


[[databases]]

[[[mysql]]]

nice_name="My SQL DB"
engine=mysql
host=192.168.0.219
port=3306
user=root
password=123456
```

## 2.重启Hue

进入hue根目录
```shell
cd /home/software/hue-4.7.1/hue
```

启动hue并挂在到后台

```shell
nohup ./build/env/bin/supervisor &
```
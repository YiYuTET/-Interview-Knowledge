# Hbase问题解决

**<font color=Red>
报错：关闭HBase时无法找到Master：no hbase master found
</font>**

<font color=OrangeRed size=5><b>原因</b></font>

此时可以大体确定报错原因，系统找不到HBase的pid文件，pid文件里面是HBase的进程号，找不到进程号系统就没有办法去结束这个进程。

HBase的pid文件默认存放路径为 /tmp 路径，可以进去看一下有没有和HBase相关的文件。肯定没有，因为很有可能被操作系统删掉了。




<font color=#0066FF size=5><b>解决方案</b></font>

修改pid文件存放路径，

进入 hbase 的 conf 目录，找到 hbase-env.sh 进行修改
```sh
export HBASE_PID_DIR=/home/software/hbase-2.3.6/pids
```
之后重新启动即可。
## Hadoop启动顺序

准备工作：

 关闭防火墙、selinux

 做时钟同步，设置active和standby两种状态下的namenode的所有节点免密登陆

 yum install psmisc。

1.HA启动顺序

```shell
首次启动顺序HDFS
1、确保配置的zookeeper服务器已经运行
2、在所有journalnode机器上启动：hdfs --daemon start journalnode
3、namenode1中执行格式化zkfc：hdfs zkfc -formatZK
4、namenode1中格式化主节点：hdfs namenode -format
5、启动namenode1中的主节点：hdfs --daemon start namenode
6、namenode2副节点同步主节点格式化：hdfs namenode -bootstrapStandby
7、启动namenode守护进程zkfc：hdfs --daemon start zkfc
8、启动所有的datanode：hdfs --daemon start datanode
9、启动mapreduce的历史信息查询服务：mapred --daemon start historyserver
首次启动顺序YARN：
1、两台：./yarn --daemon start resourcemanager
2、若干台：./yarn --daemon start nodemanager
3、一台：./yarn --daemon start timelineserver
```
2.非HA启动顺序
```shell
首次启动顺序HDFS
1.格式化namenode：hdfs namenode -format
2.启动namenode：hdfs --daemon start namenode
3.启动secondarynamenode：hdfs --daemon start secondarynamenode
4.启动datanode：hdfs --daemon start datanode
首次启动顺序YARN：
1、一台：./yarn --daemon start resourcemanager
2、若干台：./yarn --daemon start nodemanager
3、一台：./yarn --daemon start timelineserver
```
3.非HA切换到HA启动顺序
```shell
修改为HA的可用配置
1.停止namenode：hdfs --daemon stop namenode
2.停止secondarynamenode：hdfs --daemon stop secondarynamenode
3.启动zookeeper：zkServer.sh start
4.启动journalnode：hdfs --daemon start journalnode
5.备份共享日志文件：hdfs namenode -initializeSharedEdits
6.格式化namenode在zookeeper中的信息：hdfs zkfc -formatZK
7.启动namenode：hdfs --daemon start namenode
8.启动zkfc：hdfs --daemon start zkfc
9.在standby节点上格式化：hdfs namenode -bootstrapStandby
10.在standby节点上启动namenode：hdfs --daemon start namenode
11.在standby节点上启动zkfc：hdfs --daemon start zkfc
12.重启所有的datanode：hdfs --daemon stop datanode / hdfs --daemon start datanode
13.重启Yarn的所有进程：(略)
```
4.hive相关服务启动顺序
```shell
修改配置文件
1.初始化hive元数据：./schematool -dbType mysql -initSchema
2.开启metastore服务：nohup ./hive --service metastore > metastore.log 2>&1 &
3.开启hiveserver2服务：nohup ./hive --service hiveserver2 > hiveserver.log 2>&1 &
```
5.Hbase相关服务启动顺序
```shell
修改配置文件
1.主节点启动HMaster进程：./hbase-daemon.sh start master
2.从节点启动regionserver：./hbase-daemon.sh start regionserver
```

<div align="center"> <img  src="https://gitee.com/MartinHub/MartinHub-notes/raw/master/images/weixin.png" width="200"/> </div>
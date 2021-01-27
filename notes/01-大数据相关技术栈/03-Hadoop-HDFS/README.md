## 一、HDFS的使用场景

&emsp;&emsp;适合一次写入，多次读出的场景，且不支持文件的修改。适合用来做数据分析，并不适合用来做网盘应用。

## 二、HDFS  优缺点

### （1）优点

**（1）高容错性**

​        1) 数据自动保存多个副本。它通过增加副本的形式，提高容错性。

​        2) 某一个副本丢失以后，它可以自动恢复。

**（2）适合处理大数据**

​         1）数据规模：能够处理数据规模达到GB、TB、甚至PB级别的数据；

​         2）文件规模：能够处理百万规模以上的文件数量，数量相当之大。

**（3）可构建在廉价机器上，通过多副本机制，提高可靠性**

### （2）缺点

**1）不适合低延时数据访问，比如毫秒级的存储数据，是做不到的。**

**2）无法高效的对大量小文件进行存储。**

（1）存储大量小文件的话，它会占用NameNode大量的内存来存储文件

目录和块信息。这样是不可取的，因为NameNode的内存总是有限的；

（2）小文件存储的寻址时间会超过读取时间，它违反了HDFS的设计目标。

**3）不支持并发写入、文件随机修改。**

（1）一个文件只能有一个写，不允许多个线程同时写；

（2）仅支持数据append（追加），不支持文件的随机修改。

## 三、HDFS  组成架构

**1）NameNode（nn）：**就是Master，它是一个主管、管理者。

（1）管理HDFS的名称空间；

（2）配置副本策略；

（3）管理数据块（Block）映射信息；

（4）处理客户端读写请求。

**2）DataNode：**就是Slave。NameNode下达命令，DataNode执行实际的操作。

（1）存储实际的数据块；

（2）执行数据块的读/写操作。

**3）Client：**就是客户端。

（1）文件切分。文件上传HDFS的时候，Client将文件切分成一个一个的Block，然后进行上传；

（2）与NameNode交互，获取文件的位置信息；

（3）与DataNode交互，读取或者写入数据；

（4）Client提供一些命令来管理HDFS，比如NameNode格式化；

（5）Client可以通过一些命令来访问HDFS，比如对HDFS增删查改操作；

**4）Secondary NameNode：**并非NameNode的热备。当NameNode挂掉的时候，它并不

能马上替换NameNode并提供服务。

（1）辅助NameNode，分担其工作量，比如定期合并Fsimage和Edits，并推送给NameNode

（2）在紧急情况下，可辅助恢复NameNode

***HDFS 文件块大小：默认大小在Hadoop2.x版本中是128M。***

<font color='red'>思考：为什么块的大小不能设置太小 ， 也不能设置太大 ？</font>

> （1）HDFS的块设置太小，会增加寻址时间，程序一直在找块的开始位置；
>
> （2）HDFS的块比磁盘的块大，其目的是为了最小化寻址开销；
>
> （3）如果块设置的太大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间。导致程序在处理这块数据时，会非常慢。
>
> 总结：HDFS 块的大小 设置主要 取决于 磁盘传输速率。

## 四、HDFS 常用 shell 命令

```plain
[martinhub@hadoop102 hadoop-2.7.2]$ bin/hadoop fs
    [-appendToFile <localsrc> ... <dst>]
    [-cat [-ignoreCrc] <src> ...]
    [-checksum <src> ...]
    [-chgrp [-R] GROUP PATH...]
    [-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
    [-chown [-R] [OWNER][:[GROUP]] PATH...]
    [-copyFromLocal [-f] [-p] <localsrc> ... <dst>]
    [-copyToLocal [-p] [-ignoreCrc] [-crc] <src> ...
    <localdst>]
    [-count [-q] <path> ...]
    [-cp [-f] [-p] <src> ... <dst>]
    [-createSnapshot <snapshotDir> [<snapshotName>]]
    [-deleteSnapshot <snapshotDir> <snapshotName>]
    [-df [-h] [<path> ...]]
    [-du [-s] [-h] <path> ...]
    [-expunge]
    [-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
    [-getfacl [-R] <path>]
    [-getmerge [-nl] <src> <localdst>]
    [-help [cmd ...]]
    [-ls [-d] [-h] [-R] [<path> ...]]
    [-mkdir [-p] <path> ...]
    [-moveFromLocal <localsrc> ... <dst>]
    [-moveToLocal <src> <localdst>]
    [-mv <src> ... <dst>]
    [-put [-f] [-p] <localsrc> ... <dst>]
    [-renameSnapshot <snapshotDir> <oldName> <newName>]
    [-rm [-f] [-r|-R] [-skipTrash] <src> ...]
    [-rmdir [--ignore-fail-on-non-empty] <dir> ...]
    [-setfacl [-R] [{-b|-k} {-m|-x <acl_spec>} <path>]|[--
    set <acl_spec> <path>]]
    [-setrep [-R] [-w] <rep> <path> ...]
    [-stat [format] <path> ...]
    [-tail [-f] <file>]
    [-test -[defsz] <path>]
    [-text [-ignoreCrc] <src> ...]
    [-touchz <path> ...]
    [-usage [cmd ...]]
```
&emsp;&emsp;&emsp;***设置的副本数只是记录在 NameNode 的元数据中，是否真的会有这么多副本，还得***
***看 DataNode 的数量。因为目前只有 3 台设备，最多也就 3 个副本，只有节点数的增加到 10***

***台时，副本数才能达到 10。***



**1. 显示当前目录结构**

```shell
# 显示当前目录结构
hadoop fs -ls  <path>
# 递归显示当前目录结构
hadoop fs -ls  -R  <path>
# 显示根目录下内容
hadoop fs -ls  /
```

**2. 创建目录**

```shell
# 创建目录
hadoop fs -mkdir  <path> 
# 递归创建目录
hadoop fs -mkdir -p  <path>  
```

**3. 删除操作**

```shell
# 删除文件
hadoop fs -rm  <path>
# 递归删除目录和文件
hadoop fs -rm -R  <path> 
```

**4. 从本地加载文件到 HDFS**

```shell
# 二选一执行即可
hadoop fs -put  [localsrc] [dst] 
hadoop fs - copyFromLocal [localsrc] [dst] 
```

**5. 从 HDFS 导出文件到本地**

```shell
# 二选一执行即可
hadoop fs -get  [dst] [localsrc] 
hadoop fs -copyToLocal [dst] [localsrc] 
```

**6. 查看文件内容**

```shell
# 二选一执行即可
hadoop fs -text  <path> 
hadoop fs -cat  <path>  
```

**7. 显示文件的最后一千字节**

```shell
hadoop fs -tail  <path> 
# 和Linux下一样，会持续监听文件内容变化 并显示文件的最后一千字节
hadoop fs -tail -f  <path> 
```

**8. 拷贝文件**

```shell
hadoop fs -cp [src] [dst]
```

**9. 移动文件**

```shell
hadoop fs -mv [src] [dst] 
```

**10. 统计当前目录下各文件大小**  

- 默认单位字节  
- -s : 显示所有文件大小总和，
- -h : 将以更友好的方式显示文件大小（例如 64.0m 而不是 67108864）

```shell
hadoop fs -du  <path>  
```

**11. 合并下载多个文件**

- -nl  在每个文件的末尾添加换行符（LF）
- -skip-empty-file 跳过空文件

```shell
hadoop fs -getmerge
# 示例 将HDFS上的hbase-policy.xml和hbase-site.xml文件合并后下载到本地的/usr/test.xml
hadoop fs -getmerge -nl  /test/hbase-policy.xml /test/hbase-site.xml /usr/test.xml
```

**12. 统计文件系统的可用空间信息**

```shell
hadoop fs -df -h /
```

**13. 更改文件复制因子**

```shell
hadoop fs -setrep [-R] [-w] <numReplicas> <path>
```

- 更改文件的复制因子。如果 path 是目录，则更改其下所有文件的复制因子
- -w : 请求命令是否等待复制完成

```shell
# 示例
hadoop fs -setrep -w 3 /user/hadoop/dir1
```

**14. 权限控制**  

```shell
# 权限控制和Linux上使用方式一致
# 变更文件或目录的所属群组。 用户必须是文件的所有者或超级用户。
hadoop fs -chgrp [-R] GROUP URI [URI ...]
# 修改文件或目录的访问权限  用户必须是文件的所有者或超级用户。
hadoop fs -chmod [-R] <MODE[,MODE]... | OCTALMODE> URI [URI ...]
# 修改文件的拥有者  用户必须是超级用户。
hadoop fs -chown [-R] [OWNER][:[GROUP]] URI [URI ]
```

**15. 文件检测**

```shell
hadoop fs -test - [defsz]  URI
```

可选选项：

- -d：如果路径是目录，返回 0。
- -e：如果路径存在，则返回 0。
- -f：如果路径是文件，则返回 0。
- -s：如果路径不为空，则返回 0。
- -r：如果路径存在且授予读权限，则返回 0。
- -w：如果路径存在且授予写入权限，则返回 0。
- -z：如果文件长度为零，则返回 0。

```shell
# 示例
hadoop fs -test -e filename
```



## 五、HDFS 的数据流

### （1）HDFS 写数据流程

> 1）客户端通过 Distributed FileSystem 模块向 NameNode 请求上传文件，NameNode 检查目标文件是否已存在，父目录是否存在。
>
> 2）NameNode 返回是否可以上传。
>
> 3）客户端请求第一个 Block 上传到哪几个 DataNode 服务器上。
>
> 4）NameNode 返回 3 个 DataNode 节点，分别为 dn1、dn2、dn3。
>
> 5）客户端通过 FSDataOutputStream 模块请求 dn1 上传数据，dn1 收到请求会继续调用 dn2，然后 dn2 调用 dn3，将这个通信管道建立完成。
>
> 6）dn1、dn2、dn3 逐级应答客户端。
>
> 7）客户端开始往 dn1 上传第一个 Block（先从磁盘读取数据放到一个本地内存缓存），以
> Packet 为单位，dn1 收到一个 Packet 就会传给 dn2，dn2 传给 dn3；dn1 每传一个 packet 会放入一个应答队列等待应答。
>
> 8）当一个 Block 传输完成之后，客户端再次请求 NameNode 上传第二个 Block 的服务器。（重复执行 3-7 步）

### （2）HDFS 读数据流程

> 1）客户端通过 Distributed FileSystem 向 NameNode 请求下载文件，NameNode 通过查询元数据，找到文件块所在的 DataNode 地址。
>
> 2）挑选一台 DataNode（就近原则，然后随机）服务器，请求读取数据。
>
> 3）DataNode 开始传输数据给客户端（从磁盘里面读取数据输入流，以 Packet 为单位来做校验）。
>
> 4）客户端以 Packet 为单位接收，先在本地缓存，然后写入目标文件。

## 六、NameNode 和 SecondaryNameNode

### （1）NN 和 和 2NN

> 1. 第一阶段：NameNode 启动
>
>      （1）第一次启动 NameNode 格式化后，创建 Fsimage 和 Edits 文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。
>
>      （2）客户端对元数据进行增删改的请求。
>
>      （3）NameNode 记录操作日志，更新滚动日志。
>
>      （4）NameNode 在内存中对数据进行增删改。
>
> 2. 第二阶段：Secondary NameNode 工作
>
>      （1）Secondary NameNode 询问 NameNode 是否需要 CheckPoint。直接带回 NameNode 是否检查结果。
>
>      （2）Secondary NameNode 请求执行 CheckPoint。
>
>      （3）NameNode 滚动正在写的 Edits 日志。
>
>      （4）将滚动前的编辑日志和镜像文件拷贝到 Secondary NameNode。
>
>      （5）Secondary NameNode 加载编辑日志和镜像文件到内存，并合并。
>
>      （6）生成新的镜像文件 fsimage.chkpoint。
>
>      （7）拷贝 fsimage.chkpoint 到 NameNode。
>
>      （8）NameNode 将 fsimage.chkpoint 重新命名成 fsimage。

&emsp;&emsp;**NN 和 和 2NN  工作机制详解：**

&emsp;&emsp;**Fsimage：**NameNode 内存中元数据序列化后形成的文件。

&emsp;&emsp;**Edits：**记录客户端更新元数据信息的每一步操作（可通过 Edits 运算出元数据）。

### （2）NameNode 故障处理

&emsp;&emsp;NameNode 故障后，可以采用如下两种方法恢复数据。

**方法一：**将 SecondaryNameNode 中到数据拷贝到 NameNode  存储数据的目录；

1. kill -9 NameNode 进程

2. 删除 NameNode 存储的数据（/opt/module/hadoop-2.7.2/data/tmp/dfs/name）

```shell
[martinhub@hadoop102 hadoop-2.7.2]$ rm -rf /opt/module/hadoop-
2.7.2/data/tmp/dfs/name/*
```
3. 拷贝 SecondaryNameNode 中数据到原 NameNode 存储数据目录
```shell
[martinhub@hadoop102  dfs]$  scp  -r
martinhub@hadoop104:/opt/module/hadoop-2.7.2/data/tmp/dfs/namesecondary/* ./name/
```
4. 重新启动 NameNode
```shell
[martinhub@hadoop102 hadoop-2.7.2]$ sbin/hadoop-daemon.sh start namenode
```
**方法 二 ：**使用 -importCheckpoint  选项启动 NameNode 守护进程， 从 而 将
SecondaryNameNode 中数据拷贝到NameNode 目录中。

1. 修改 hdfs-site.xml 中的

```shell
<property>
  <name>dfs.namenode.checkpoint.period</name>
  <value>120</value>
</property>
<property>
  <name>dfs.namenode.name.dir</name>
  <value>/opt/module/hadoop-2.7.2/data/tmp/dfs/name</value>
</property>
```
2. kill -9 NameNode 进程

3. 删除 NameNode 存储的数据（/opt/module/hadoop-2.7.2/data/tmp/dfs/name）

```shell
[martinhub@hadoop102 hadoop-2.7.2]$ rm -rf /opt/module/hadoop-
2.7.2/data/tmp/dfs/name/*
```
4. 如果 SecondaryNameNode 不和 NameNode 在一个主机节点上，需要将

SecondaryNameNode 存储数据的目录拷贝到 NameNode 存储数据的平级目录，并删

除 in_use.lock 文件

```shell
[martinhub@hadoop102  dfs]$  scp  -r
martinhub@hadoop104:/opt/module/hadoop-
2.7.2/data/tmp/dfs/namesecondary ./
[martinhub@hadoop102 namesecondary]$ rm -rf in_use.lock
[martinhub@hadoop102 dfs]$ pwd
/opt/module/hadoop-2.7.2/data/tmp/dfs
[martinhub@hadoop102 dfs]$ ls
data name namesecondary
```
5. 导入检查点数据（等待一会 ctrl+c 结束掉）
```shell
[martinhub@hadoop102  hadoop-2.7.2]$  bin/hdfs  namenode  -importCheckpoint
```
6. 启动 NameNode
```shell
[martinhub@hadoop102 hadoop-2.7.2]$ sbin/hadoop-daemon.sh start namenode
```
## 七、Datanode工作机制

&emsp;&emsp;1）一个数据块在 DataNode 上以文件形式存储在磁盘上，包括两个文件，一个是数据本

身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。

&emsp;&emsp;2）DataNode 启动后向 NameNode 注册，通过后，周期性（1 小时）的向 NameNode 上

报所有的块信息。

&emsp;&emsp;3）心跳是每 3 秒一次，心跳返回结果带有 NameNode 给该 DataNode 的命令如复制块数

据到另一台机器，或删除某个数据块。如果超过 10 分钟没有收到某个 DataNode 的心跳，则

认为该节点不可用。

&emsp;&emsp;4）集群运行中可以安全加入和退出一些机器。


## 一、Zookeeper简介

​	&emsp;&emsp;Zookeeper 是一个开源的分布式的，为分布式应用提供协调服务的 Apache 项目。

## 二、Zookerrper的工作机制

&emsp;&emsp;Zookeeper从设计模式角度来理解：是一个基于观察者模式设计的分布式服务管理框架，它负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦这些数据的状态发生变化，Zookeeper就将负责通知已经在Zookeeper上注册的那些观察者做出相应的反应。

## 三、Zookeeper的特点

1）Zookeeper：一个领导者（Leader），多个跟随者（Follower）组成的集群。

2）集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。

3）全局数据一致：每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。

4）更新请求顺序进行，来自同一个Client的更新请求按其发送顺序依次执行。

5）数据更新原子性，一次数据更新要么成功，要么失败。

6）实时性，在一定时间范围内，Client能读到最新数据。

## 四、应用场景

&emsp;&emsp;提供的服务包括：统一命名服务、统一配置管理、统一集群管理、服务器节点动态上下线、软负载均衡等。

### （1）统一命名服务

&emsp;&emsp;在分布式环境下，经常需要对应用/服务进行统一命名，便于识别。例如：IP不容易记住，而域名容易记住。

### （2）统一配置管理

&emsp;1）分布式环境下，配置文件同步非常常见。

&emsp;（1）一般要求一个集群中，所有节点的配置信息是一致的，比如 Kafka 集群。

&emsp;（2）对配置文件修改后，希望能够快速同步到各个节点上。

&emsp;2）配置管理可交由ZooKeeper实现。

&emsp;（1）可将配置信息写入ZooKeeper上的一个Znode。

&emsp;（2）各个客户端服务器监听这个Znode。

&emsp;（3）一旦Znode中的数据被修改，ZooKeeper将通知各个客户端服务器。

### （3）统一集群管理

&emsp;1）分布式环境中，实时掌握每个节点的状态是必要的。

&emsp;（1）可根据节点实时状态做出一些调整。

&emsp;2）ZooKeeper可以实现实时监控节点状态变化

&emsp;（1）可将节点信息写入ZooKeeper上的一个ZNode。

&emsp;（2）监听这个ZNode可获取它的实时状态变化。

### （4）服务器动态上下线

&emsp;客户端能实时洞察到服务器上下线的变化。

### （5）软负载均衡

&emsp;在Zookeeper中记录每台服务器的访问数，让访问数最少的服务器去处理最新的客户端请求。

## 五、Zookeeper的内部原理

### （1）选举机制

&emsp;1）半数机制：集群中半数以上机器存活，集群可用。所以 Zookeeper 适合安装奇数台服务器。

&emsp;2）Zookeeper 虽然在配置文件中并没有指定 Master 和 Slave。但是，Zookeeper 工作时，是有一个节点为Leader，其他则为 Follower，Leader 是通过内部的选举机制临时产生的。



## 六、Zookeeper单机环境搭建

### （1）下载

下载对应版本 Zookeeper，这里我下载的版本 `3.4.14`。官方下载地址：https://archive.apache.org/dist/zookeeper/

```shell
# wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz
```

### （2）解压

```shell
# tar -zxvf zookeeper-3.4.14.tar.gz
```

### （3）配置环境变量

```shell
# vim /etc/profile
```

添加环境变量：

```shell
export ZOOKEEPER_HOME=/usr/app/zookeeper-3.4.14
export PATH=$ZOOKEEPER_HOME/bin:$PATH
```

使得配置的环境变量生效：

```shell
# source /etc/profile
```

### （4）修改配置

进入安装目录的 `conf/` 目录下，拷贝配置样本并进行修改：

```
# cp zoo_sample.cfg  zoo.cfg
```

指定数据存储目录和日志文件目录（目录不用预先创建，程序会自动创建），修改后完整配置如下：

```properties
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/usr/local/zookeeper/data
dataLogDir=/usr/local/zookeeper/log
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```

> 配置参数说明：
>
> - **tickTime**：用于计算的基础时间单元。比如 session 超时：N*tickTime；
> - **initLimit**：用于集群，允许从节点连接并同步到 master 节点的初始化连接时间，以 tickTime 的倍数来表示；
> - **syncLimit**：用于集群， master 主节点与从节点之间发送消息，请求和应答时间长度（心跳机制）；
> - **dataDir**：数据存储位置；
> - **dataLogDir**：日志目录；
> - **clientPort**：用于客户端连接的端口，默认 2181



### （5）启动

由于已经配置过环境变量，直接使用下面命令启动即可：

```
zkServer.sh start
```

### （6）验证

使用 JPS 验证进程是否已经启动，出现 `QuorumPeerMain` 则代表启动成功。

```shell
[root@hadoop001 bin]# jps
3814 QuorumPeerMain
```



## 七、Zookeeper集群环境搭建

&emsp;&emsp;为保证集群高可用，Zookeeper 集群的节点数最好是奇数，最少有三个节点，所以这里演示搭建一个三个节点的集群。这里我使用三台主机进行搭建，主机名分别为 hadoop101，hadoop102，hadoop103。

### （1）zoo.cfg 参数解读

&emsp;&emsp;Zookeeper中的配置文件 zoo.cfg 中参数含义解读如下：

```text
1．tickTime =2000：通信心跳数，Zookeeper 服务器与客户端心跳时间，单位毫秒
	Zookeeper使用的基本时间，服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个tickTime时间就会发送一个心跳，时间单位为毫秒。
	它用于心跳机制，并且设置最小的session超时时间为两倍心跳时间。(session的最小超时时间是2*tickTime)
	
2．initLimit =10：LF 初始通信时限
	集群中的Follower跟随者服务器与Leader领导者服务器之间初始连接时能容忍的最多心跳数（tickTime的数量），用它来限定集群中的Zookeeper服务器连接到Leader的时限。
	
3．syncLimit =5：LF 同步通信时限
	集群中Leader与Follower之间的最大响应时间单位，假如响应超过syncLimit * tickTime，Leader认为Follwer死掉，从服务器列表中删除Follwer。

4．dataDir：数据文件目录+数据持久化路径
	主要用于保存 Zookeeper 中的数据。
5．clientPort =2181：客户端连接端口
	监听客户端连接的端口。
```

### （2）修改配置

&emsp;&emsp;解压一份 zookeeper 安装包，修改其配置文件 `zoo.cfg`，内容如下。之后使用 scp 命令将安装包分发到三台服务器上：

```shell
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/usr/local/zookeeper-cluster/data/
dataLogDir=/usr/local/zookeeper-cluster/log/
clientPort=2181

# server.1 这个1是服务器的标识，可以是任意有效数字，标识这是第几个服务器节点，这个标识要写到dataDir目录下面myid文件里
# 指名集群间通讯端口和选举端口
server.1=hadoop101:2287:3387
server.2=hadoop102:2287:3387
server.3=hadoop103:2287:3387
```

### （3）标识节点

&emsp;&emsp;分别在三台主机的 `dataDir` 目录下新建 `myid` 文件,并写入对应的节点标识。Zookeeper 集群通过 `myid` 文件识别集群节点，并通过上文配置的节点通信端口和选举端口来进行节点通信，选举出 Leader 节点。

&emsp;&emsp;创建存储目录：

```shell
# 三台主机均执行该命令
mkdir -vp  /usr/local/zookeeper-cluster/data/
```

&emsp;&emsp;创建并写入节点标识到 `myid` 文件：

```shell
# hadoop001主机
echo "1" > /usr/local/zookeeper-cluster/data/myid
# hadoop002主机
echo "2" > /usr/local/zookeeper-cluster/data/myid
# hadoop003主机
echo "3" > /usr/local/zookeeper-cluster/data/myid
```

### （4）启动集群

&emsp;&emsp;分别在三台主机上，执行如下命令启动服务：

```shell
./bin/zkServer.sh start
```

### （5）集群验证

&emsp;&emsp;启动后使用 `zkServer.sh status` 查看集群各个节点状态。如图所示：三个节点进程均启动成功，并且 hadoop102 为 leader 节点，hadoop101 和 hadoop103 为 follower 节点。

```shell
[MartinHub@hadoop101 zookeeper-3.4.10]# bin/zkServer.sh status
	JMX enabled by default Using  config:  /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg Mode: fsollower
	
[MartinHub@hadoop102 zookeeper-3.4.10]# bin/zkServer.sh status
	JMX enabled by default Using  config:  /opt/module/zookeeper- 3.4.10/bin/../conf/zoo.cfg Mode: leader

[MartinHub@hadoop103 zookeeper-3.4.5]# bin/zkServer.sh status
	JMX enabled by default Using  config:  /opt/module/zookeeper- 3.4.10/bin/../conf/zoo.cfg Mode: follower
```

## 八、Zookeeper常用shell命令

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/01-Zookeeper/images/zook1.PNG)

### （1）节点增删改查

#### > 启动服务和连接服务

```shell
# 启动服务
bin/zkServer.sh start

#连接服务 不指定服务地址则默认连接到localhost:2181
zkCli.sh -server hadoop001:2181
```

#### > help命令

&emsp;&emsp;使用 `help` 可以查看所有命令及格式。

#### > 查看节点列表

&emsp;&emsp;查看节点列表有 `ls path` 和 `ls2 path` 两个命令，后者是前者的增强，不仅可以查看指定路径下的所有节点，还可以查看当前节点的信息。

```shell
[zk: localhost:2181(CONNECTED) 0] ls /
[cluster, controller_epoch, brokers, storm, zookeeper, admin,  ...]
[zk: localhost:2181(CONNECTED) 1] ls2 /
[cluster, controller_epoch, brokers, storm, zookeeper, admin, ....]
cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x130
cversion = 19
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 11
```

#### > 新增节点

```shell
create [-s] [-e] path data acl   #其中-s 为有序节点，-e 临时节点
```

&emsp;&emsp;创建节点并写入数据：

```shell
create /hadoop 123456
```

&emsp;&emsp;创建有序节点，此时创建的节点名为指定节点名 + 自增序号：

```shell
[zk: localhost:2181(CONNECTED) 23] create -s /a  "aaa"
Created /a0000000022
[zk: localhost:2181(CONNECTED) 24] create -s /b  "bbb"
Created /b0000000023
[zk: localhost:2181(CONNECTED) 25] create -s /c  "ccc"
Created /c0000000024
```

&emsp;&emsp;创建临时节点，临时节点会在会话过期后被删除：

```shell
[zk: localhost:2181(CONNECTED) 26] create -e /tmp  "tmp"
Created /tmp
```

#### > 查看节点

1. 获取节点数据

```shell
# 格式
get path [watch] 
```

```shell
[zk: localhost:2181(CONNECTED) 31] get /hadoop
123456   #节点数据
cZxid = 0x14b
ctime = Fri May 24 17:03:06 CST 2019
mZxid = 0x14b
mtime = Fri May 24 17:03:06 CST 2019
pZxid = 0x14b
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
```

&emsp;&emsp;节点各个属性如下表。其中一个重要的概念是 Zxid(ZooKeeper Transaction  Id)，ZooKeeper 节点的每一次更改都具有唯一的 Zxid，如果 Zxid1 小于 Zxid2，则 Zxid1 的更改发生在 Zxid2 更改之前。

| **状态属性**       | **说明**                                   |
| -------------- | ---------------------------------------- |
| cZxid          | 数据节点创建时的事务 ID                            |
| ctime          | 数据节点创建时的时间                               |
| mZxid          | 数据节点最后一次更新时的事务 ID                        |
| mtime          | 数据节点最后一次更新时的时间                           |
| pZxid          | 数据节点的子节点最后一次被修改时的事务 ID                   |
| cversion       | 子节点的更改次数                                 |
| dataVersion    | 节点数据的更改次数                                |
| aclVersion     | 节点的 ACL 的更改次数                            |
| ephemeralOwner | 如果节点是临时节点，则表示创建该节点的会话的 SessionID；如果节点是持久节点，则该属性值为 0 |
| dataLength     | 数据内容的长度                                  |
| numChildren    | 数据节点当前的子节点个数                             |

2. 查看节点状态

&emsp;&emsp;可以使用 `stat` 命令查看节点状态，它的返回值和 `get` 命令类似，但不会返回节点数据。

```shell
[zk: localhost:2181(CONNECTED) 32] stat /hadoop
cZxid = 0x14b
ctime = Fri May 24 17:03:06 CST 2019
mZxid = 0x14b
mtime = Fri May 24 17:03:06 CST 2019
pZxid = 0x14b
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
```

#### > 更新节点

&emsp;&emsp;更新节点的命令是 `set`，可以直接进行修改，如下：

```shell
[zk: localhost:2181(CONNECTED) 33] set /hadoop 345
cZxid = 0x14b
ctime = Fri May 24 17:03:06 CST 2019
mZxid = 0x14c
mtime = Fri May 24 17:13:05 CST 2019
pZxid = 0x14b
cversion = 0
dataVersion = 1  # 注意更改后此时版本号为 1，默认创建时为 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
```

&emsp;&emsp;也可以基于版本号进行更改，此时类似于乐观锁机制，当你传入的数据版本号 (dataVersion) 和当前节点的数据版本号不符合时，zookeeper 会拒绝本次修改：

```shell
[zk: localhost:2181(CONNECTED) 34] set /hadoop 678 0
version No is not valid : /hadoop    #无效的版本号
```

#### > 删除节点

&emsp;&emsp;删除节点的语法如下：

```shell
delete path [version]
```

&emsp;&emsp;和更新节点数据一样，也可以传入版本号，当你传入的数据版本号 (dataVersion) 和当前节点的数据版本号不符合时，zookeeper 不会执行删除操作。

```shell
[zk: localhost:2181(CONNECTED) 36] delete /hadoop 0
version No is not valid : /hadoop   #无效的版本号
[zk: localhost:2181(CONNECTED) 37] delete /hadoop 1
[zk: localhost:2181(CONNECTED) 38]
```

&emsp;&emsp;要想删除某个节点及其所有后代节点，可以使用递归删除，命令为 `rmr path`。

### （2）监听器

#### > get path [watch]

&emsp;&emsp;使用 `get path [watch]` 注册的监听器能够在节点内容发生改变的时候，向客户端发出通知。需要注意的是 zookeeper 的触发器是一次性的 (One-time trigger)，即触发一次后就会立即失效。

```shell
[zk: localhost:2181(CONNECTED) 4] get /hadoop  watch
[zk: localhost:2181(CONNECTED) 5] set /hadoop 45678
WATCHER::
WatchedEvent state:SyncConnected type:NodeDataChanged path:/hadoop  #节点值改变
```

#### > stat path [watch]

&emsp;&emsp;使用 `stat path [watch]` 注册的监听器能够在节点状态发生改变的时候，向客户端发出通知。

```shell
[zk: localhost:2181(CONNECTED) 7] stat /hadoop watch
[zk: localhost:2181(CONNECTED) 8] set /hadoop 112233
WATCHER::
WatchedEvent state:SyncConnected type:NodeDataChanged path:/hadoop  #节点值改变
```

#### > ls\ls2 path  [watch]

&emsp;&emsp;使用 `ls path [watch]` 或 `ls2 path [watch]` 注册的监听器能够监听该节点下所有**子节点**的增加和删除操作。

```shell
[zk: localhost:2181(CONNECTED) 9] ls /hadoop watch
[]
[zk: localhost:2181(CONNECTED) 10] create  /hadoop/yarn "aaa"
WATCHER::
WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/hadoop
```



### （3）zookeeper 四字命令

| 命令   | 功能描述                                     |
| ---- | ---------------------------------------- |
| conf | 打印服务配置的详细信息。                             |
| cons | 列出连接到此服务器的所有客户端的完整连接/会话详细信息。包括接收/发送的数据包数量，会话 ID，操作延迟，上次执行的操作等信息。 |
| dump | 列出未完成的会话和临时节点。这只适用于 Leader 节点。           |
| envi | 打印服务环境的详细信息。                             |
| ruok | 测试服务是否处于正确状态。如果正确则返回“imok”，否则不做任何相应。     |
| stat | 列出服务器和连接客户端的简要详细信息。                      |
| wchs | 列出所有 watch 的简单信息。                        |
| wchc | 按会话列出服务器 watch 的详细信息。                    |
| wchp | 按路径列出服务器 watch 的详细信息。                    |

> 更多四字命令可以参阅官方文档：https://zookeeper.apache.org/doc/current/zookeeperAdmin.html

&emsp;&emsp;使用前需要使用 `yum install nc` 安装 nc 命令，使用示例如下：

```shell
[root@hadoop001 bin]# echo stat | nc localhost 2181
Zookeeper version: 3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, 
built on 06/29/2018 04:05 GMT
Clients:
 /0:0:0:0:0:0:0:1:50584[1](queued=0,recved=371,sent=371)
 /0:0:0:0:0:0:0:1:50656[0](queued=0,recved=1,sent=0)
Latency min/avg/max: 0/0/19
Received: 372
Sent: 371
Connections: 2
Outstanding: 0
Zxid: 0x150
Mode: standalone
Node count: 167
```

## 九、企业面试题

- 请简述 ZooKeeper  的选举机制

- ZooKeeper  的监听原理是什么？

- ZooKeeper  的部署方式有哪几种？集群中的角色有哪些？集群最少需要几台机器？

  （1）部署方式单机模式、集群模式

  （2）角色：Leader 和 Follower

  （3）集群最少需要机器数：3




<div align="center"> <img  src="https://gitee.com/MartinHub/MartinHub-notes/raw/master/images/weixin.png" width="200"/> </div>















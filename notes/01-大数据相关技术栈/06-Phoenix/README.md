![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/06-Phoenix/images/phoenix.PNG)



## 一、Phoenix的简介

### （1）定义 

&emsp;&emsp;`Phoenix` 是 HBase 的开源 SQL 中间层，它允许你使用标准 JDBC 的方式来操作 HBase 上的数据。在 `Phoenix` 之前，如果你要访问 HBase，只能调用它的 Java API，但相比于使用一行 SQL 就能实现数据查询，HBase 的 API 还是过于复杂。`Phoenix` 的理念是 `we put sql SQL back in NOSQL`，即你可以使用标准的 SQL 就能完成对 HBase 上数据的操作。同时这也意味着你可以通过集成 `Spring Data  JPA` 或 `Mybatis` 等常用的持久层框架来操作 HBase。

&emsp;&emsp;其次 `Phoenix` 的性能表现也非常优异，`Phoenix` 查询引擎会将 SQL 查询转换为一个或多个 HBase Scan，通过并行执行来生成标准的 JDBC 结果集。它通过直接使用 HBase API 以及协处理器和自定义过滤器，可以为小型数据查询提供毫秒级的性能，为千万行数据的查询提供秒级的性能。同时 Phoenix 还拥有二级索引等 HBase 不具备的特性，因为以上的优点，所以 `Phoenix` 成为了 HBase 最优秀的 SQL 中间层。

### （2）特点

```text
1 将SQL查询编译为HBase扫描
2 确定扫描 Rowkey 的最佳开始和结束位置
3 扫描并行执行
4 将 where 子句推送到服务器端的过滤器
5 通过协处理器进行聚合操作
6 完美支持 HBase 二级索引创建
7 DML 命令以及通过 DDL 命令创建和操作表和版本化增量更改。
8 容易集成：如 Spark，Hive，Pig，Flume 和 Map Reduce。
```

### （3）Phoenix架构

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/06-Phoenix/images/架构.PNG)

### （4）Phoenix数据存储

&emsp;&emsp;phoenix将HBase数据模型映射到关系型世界。

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/06-Phoenix/images/sql.PNG)

## 二、Phoenix入门

### （1）安装部署

> 我们可以按照官方安装说明进行安装，官方说明如下：
>
> - download and expand our installation tar
> - copy the phoenix server jar that is compatible with your HBase installation into the lib directory of every region server
> - restart the region servers
> - add the phoenix client jar to the classpath of your HBase client
> - download and setup SQuirrel as your SQL client so you can issue adhoc SQL against your HBase cluster

**下载并解压** 

&emsp;&emsp;官方针对 Apache 版本和 CDH 版本的 HBase 均提供了安装包，按需下载即可。官方下载地址: http://phoenix.apache.org/download.html

```shell
# 下载
wget http://mirror.bit.edu.cn/apache/phoenix/apache-phoenix-4.14.0-cdh5.14.2/bin/apache-phoenix-4.14.0-cdh5.14.2-bin.tar.gz
# 解压
tar tar apache-phoenix-4.14.0-cdh5.14.2-bin.tar.gz
```

**拷贝Jar包**

&emsp;&emsp;按照官方文档的说明，需要将 `phoenix server jar` 添加到所有 `Region Servers` 的安装目录的 `lib` 目录下。

&emsp;&emsp;这里由于我搭建的是 HBase 伪集群，所以只需要拷贝到当前机器的 HBase 的 lib 目录下。如果是真实集群，则使用 scp 命令分发到所有 `Region Servers` 机器上。

```shell
cp /usr/app/apache-phoenix-4.14.0-cdh5.14.2-bin/phoenix-4.14.0-cdh5.14.2-server.jar /usr/app/hbase-1.2.0-cdh5.15.2/lib
```

**重启 Region Servers**

```shell
# 停止Hbase
stop-hbase.sh
# 启动Hbase
start-hbase.sh
```

**启动Phoenix**

&emsp;&emsp;在 Phoenix 解压目录下的 `bin` 目录下执行如下命令，需要指定 Zookeeper 的地址：

- 如果 HBase 采用 Standalone 模式或者伪集群模式搭建，则默认采用内置的 Zookeeper 服务，端口为 2181；
- 如果是 HBase 是集群模式并采用外置的 Zookeeper 集群，则按照自己的实际情况进行指定。

```shell
# ./sqlline.py hadoop001:2181
```

**启动结果**

&emsp;&emsp;启动后则进入了 Phoenix 交互式 SQL 命令行，可以使用 `!table` 或 `!tables` 查看当前所有表的信息

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/06-Phoenix/images/phoenix-shell.png) 

### （2）Phoenix表操作

#### > 创建表

```sql
CREATE TABLE IF NOT EXISTS us_population (
      state CHAR(2) NOT NULL,
      city VARCHAR NOT NULL,
      population BIGINT
      CONSTRAINT my_pk PRIMARY KEY (state, city));
```

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/06-Phoenix/images/Phoenix-create-table.png)
新建的表会按照特定的规则转换为 HBase 上的表，关于表的信息，可以通过 Hbase Web UI 进行查看：

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/06-Phoenix/images/hbase-web-ui-phoenix.png)

#### > 插入数据

Phoenix 中插入数据采用的是 `UPSERT` 而不是 `INSERT`,因为 Phoenix 并没有更新操作，插入相同主键的数据就视为更新，所以 `UPSERT` 就相当于 `UPDATE`+`INSERT`

```shell
UPSERT INTO us_population VALUES('NY','New York',8143197);
UPSERT INTO us_population VALUES('CA','Los Angeles',3844829);
UPSERT INTO us_population VALUES('IL','Chicago',2842518);
UPSERT INTO us_population VALUES('TX','Houston',2016582);
UPSERT INTO us_population VALUES('PA','Philadelphia',1463281);
UPSERT INTO us_population VALUES('AZ','Phoenix',1461575);
UPSERT INTO us_population VALUES('TX','San Antonio',1256509);
UPSERT INTO us_population VALUES('CA','San Diego',1255540);
UPSERT INTO us_population VALUES('TX','Dallas',1213825);
UPSERT INTO us_population VALUES('CA','San Jose',912332);
```

#### > 修改数据

```sql
-- 插入主键相同的数据就视为更新
UPSERT INTO us_population VALUES('NY','New York',999999);
```

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/06-Phoenix/images/Phoenix-update.png)

#### > 删除数据

```sql
DELETE FROM us_population WHERE city='Dallas';
```

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/06-Phoenix/images/Phoenix-delete.png)

#### > 查询数据

```sql
SELECT state as "州",count(city) as "市",sum(population) as "热度"
FROM us_population
GROUP BY state
ORDER BY sum(population) DESC;
```

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/06-Phoenix/images/Phoenix-select.png)

#### > 退出命令

```sql
!quit
```

### （3）Phoenix 创建二级索引

#### > HBase的二级索引

&emsp;&emsp;我们知道 HBase 只能通过 rowkey 进行搜索, 一般把 rowkey 称作一级索引. 在很长的一段时间里 HBase 就只支持一级索引。HBase 里面只有 rowkey 作为一级索引， 如果要对库里的非 rowkey 字段进行数据检索和查询， 往往要通过 MapReduce/Spark 等分布式计算框架进行，硬件资源消耗和时间延迟都会比较高。为了 HBase 的数据查询更高效、适应更多的场景， 诸如使用非 rowkey 字段检索也能做到秒级响应，或者支持各个字段进行模糊查询和多字段组合查询等， 因此需要在 HBase 上面构建二级索引， 以满足现实中更复杂多样的业务需求。从 0.94 版本开始, HBase 开始支持二级索引.HBase 索引有多种放方案，我们今天要做的是使用 Phoenix 给 HBase 添加二级索引。

#### > 配置 HBase  支持 Phoenix  创建二级索引

需要先给 HBase 配置支持创建二级索引
步骤 1:  添加如下配置到 HBase  的 Hregionerver  节点的 hbase-site.xml

```shell
<!-- phoenix regionserver  配置参数 -->
<property>
	<name>hbase.regionserver.wal.codec</name>
	<value>org.apache.hadoop.hbase.regionserver.wal.IndexedWALEditCodec</value>
</property>

<property>
	<name>hbase.region.server.rpc.scheduler.factory.class</name>
	<value>org.apache.hadoop.hbase.ipc.PhoenixRpcSchedulerFactory</value>
	<description>Factory to create the Phoenix RPC Scheduler that uses separate queues for index and metadata updates</description>
</property>

<property>
	<name>hbase.rpc.controllerfactory.class</name>
	<value>org.apache.hadoop.hbase.ipc.controller.ServerRpcControllerFactory</value>
	<description>Factory to create the Phoenix RPC Scheduler that usesseparate queues for index and metadata updates</description>
</property>
```

步 骤 2:  添加如下配置到 HBase  的 Hmaster  节点的 hbase-site.xml

```shell
<!-- phoenix master  配置参数 -->
<property>
	<name>hbase.master.loadbalancer.class</name>
	<value>org.apache.phoenix.hbase.index.balancer.IndexLoadBalancer</value>
</property>

<property>
	<name>hbase.coprocessor.master.classes</name>
	<value>org.apache.phoenix.hbase.index.master.IndexMasterObserver</value>
</property>
```

步骤 3:  测试是否支持

准备数据:

```sql
create table user_1(id varchar primary key, name varchar, addr varchar)
upsert into user_1 values ('1', 'zs', 'beijing');
upsert into user_1 values ('2', 'lisi', 'shanghai');
upsert into user_1 values ('3', 'ww', 'sz');
```

默认情况下, 只要 rowkey 支持索引(就是上面的 id)

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/06-Phoenix/images/index1.PNG)

其他字段是不支持索引的:

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/06-Phoenix/images/index2.PNG)

给 name 字段添加索引:

```sql
create index idx_user_1 on user_1(name)
```

注意: 这种索引, 对  name 创建的索引, 则查询的时候也必须只查询  name 字段.

#### > Phoenix  创建索引

**Phoenix  索引分类**

&emsp;&emsp;Phoenix 索引分全局索引和局部索引
（1）全局索引

```sql
global index 是默认的索引格式。
适用于多读少写的业务场景。写数据的时候会消耗大量开销，因为索引表也要更新，而索引表是分布在不同的数据节点上的，跨节点的数据传输带来了较大的性能消耗。在读数据的时候 Phoenix 会选择索引表来降低查询消耗的时间。如果想查询的字段不是索引字段的话索引表不会被使用，也就是说不会带来查询速度的提升。

创建全局索引的方法:	
	CREATE INDEX my_index ON my_table (my_col)
```

（2）局部索引

```sql
local index 适用于写操作频繁的场景。索引数据和数据表的数据是存放在相同的服务器中的，避免了在写操作的时候往不同服务器的索引表中写索引带来的额外开销。查询的字段不是索引字段索引表也会被使用，这会带来查询速度的提升。

创建局部索引的方法(相比全局索引多了一个关键字 local):
CREATE LOCAL INDEX my_index ON my_table (my_index)
```

<font color='red'>Local index  和 Global index 区别：</font>
Local index 由于是数据与索引在同一服务器上，所以要查询的数据在哪台服务器的哪个 region 是无法定位的，只能先找到 region 然后再利用索引。

Global index 是一种分布式索引，可以直接利用索引定位服务器和 region，速度更快，但是由于分布式的原因，数据一旦出现新增变化，分布式的索引要进行跨服务的同步操作，带来大量通信消耗，所以在频繁操作的字段不适合创建全局索引。

## 三、扩展

从上面的操作中可以看出，Phoenix 支持大多数标准的 SQL 语法。关于 Phoenix 支持的语法、数据类型、函数、序列等详细信息，因为涉及内容很多，可以参考其官方文档，官方文档上有详细的说明：

- **语法 (Grammar)** ：https://phoenix.apache.org/language/index.html
- **函数 (Functions)** ：http://phoenix.apache.org/language/functions.html
- **数据类型 (Datatypes)** ：http://phoenix.apache.org/language/datatypes.html
- **序列 (Sequences)** :http://phoenix.apache.org/sequences.html
- **联结查询 (Joins)** ：http://phoenix.apache.org/joins.html






<div align="center"> <img  src="https://gitee.com/MartinHub/MartinHub-notes/raw/master/images/weixin.png" width="200"/> </div>















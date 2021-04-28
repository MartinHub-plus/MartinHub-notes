## 一、Presto 简单介绍

### （1）Presto基本概念

&emsp;&emsp;Presto是Facebook开源的MPP SQL引擎，旨在填补Hive在速度和灵活性（对接多种数据源）上的不足。相似的SQL on Hadoop竞品还有Impala和Spark SQL等。这里我们介绍下Presto的基本概念。

&emsp;&emsp;Presto是一个分布式的查询引擎，本身并不存储数据，但是可以接入多种数据源，并且支持跨数据源的级联查询。Presto是一个OLAP的工具，擅长对海量数据进行复杂的分析；但是对于OLTP场景，并不是Presto所擅长，所以不要把Presto当做数据库来使用。

&emsp;&emsp;和大家熟悉的Mysql相比：首先Mysql是一个数据库，具有存储和计算分析能力，而Presto只有计算分析能力；其次数据量方面，Mysql作为传统单点关系型数据库不能满足当前大数据量的需求，于是有各种大数据的存储和分析工具产生，Presto就是这样一个可以满足大数据量分析计算需求的一个工具。

### （2）数据源

&emsp;&emsp;Presto需要从其他数据源获取数据来进行运算分析，它可以连接多种数据源，包括Hive、RDBMS（Mysql、Oracle、Tidb等）、Kafka、MongoDB、Redis等。

&emsp;&emsp;一条Presto查询可以将多个数据源的数据进行合并分析。

&emsp;&emsp;比如：select * from a join b where a.id=b.id;，其中表a可以来自Hive，表b可以来自Mysql。

### （3）Presto架构

![img](src="https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/19-Hudi/images/架构.png)

### （4）Presto 优缺点

![img](src="https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/19-Hudi/images/优缺点.png)

### （5）Presto、Impala性能比较

     https://blog.csdn.net/u012551524/article/details/79124532

&emsp;&emsp;测试结论：Impala性能稍领先于Presto，但是Presto在数据源支持上非常丰富，包括Hive、图数据库、传统关系型数据库、Redis等。

<div align="center"> <img  src="https://gitee.com/MartinHub/MartinHub-notes/raw/master/images/weixin.png" width="200"/> </div>

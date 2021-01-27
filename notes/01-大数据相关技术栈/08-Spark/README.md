![img](./images/spark.jpg)



## 一、Spark-Core

### （1）Spark 概述

> Spark 是一种基于内存的快速、通用、可扩展的大数据分析计算引擎。
>
> Spark和Hadoop的根本差异是多个作业之间的数据通信问题 : 
>
> &emsp;Spark多个作业之间数据通信是基于内存，而 Hadoop 是基于磁盘。
>
> Spark 只有在 shuffle 的时候将数据写入磁盘，而 Hadoop 中多个 MR 作业之间的数据交互都要依赖于磁盘交互。
>
> &emsp;在绝大多数的数据计算场景中，Spark 确实会比 MapReduce更有优势。但是 Spark 是基于内存的，所以在实际的生产环境中，由于内存的限制，可能会由于内存资源不够导致 Job 执行失败，此时，MapReduce 其实是一个更好的选择，所以 Spark并不能完全替代 MR。

### （2）Spark的核心模块

![img](./images/模块.PNG)

➢  Spark Core

&emsp;Spark Core 中提供了 Spark 最基础与最核心的功能，Spark 其他的功能如：Spark SQL，Spark Streaming，GraphX, MLlib 都是在 Spark Core 的基础上进行扩展的

➢  Spark SQL

&emsp;Spark SQL 是 Spark 用来操作结构化数据的组件。通过 Spark SQL，用户可以使用 SQL或者 Apache Hive 版本的 SQL 方言（HQL）来查询数据。

➢  Spark Streaming

&emsp;Spark Streaming 是 Spark 平台上针对实时数据进行流式计算的组件，提供了丰富的处理数据流的 API。

➢  Spark MLlib

&emsp;MLlib 是 Spark 提供的一个机器学习算法库。MLlib 不仅提供了模型评估、数据导入等额外的功能，还提供了一些更底层的机器学习原语。

➢  Spark GraphX

&emsp;GraphX 是 Spark 面向图计算提供的框架与算法库。

### （3）Spark开发环境搭建

#### > 安装Spark

**下载并解压**

官方下载地址：http://spark.apache.org/downloads.html ，选择 Spark 版本和对应的 Hadoop 版本后再下载：

![img](./images/spark-download.png)

解压安装包：

```shell
tar -zxvf  spark-2.2.3-bin-hadoop2.6.tgz
```

**配置环境变量**

```shell
vim /etc/profile
```

添加环境变量：

```shell
export SPARK_HOME=/usr/app/spark-2.2.3-bin-hadoop2.6
export  PATH=${SPARK_HOME}/bin:$PATH
```

使得配置的环境变量立即生效：

```shell
source /etc/profile
```

**Local模式**

Local 模式是最简单的一种运行方式，它采用单节点多线程方式运行，不用部署，开箱即用，适合日常测试开发。

```shell
# 启动spark-shell
spark-shell --master local[2]
```

- **local**：只启动一个工作线程；
- **local[k]**：启动 k 个工作线程；
- **local[*]**：启动跟 cpu 数目相同的工作线程数。

![img](./images/spark-shell-local.png)

进入 spark-shell 后，程序已经自动创建好了上下文 `SparkContext`，等效于执行了下面的 Scala 代码：

```java
val conf = new SparkConf().setAppName("Spark shell").setMaster("local[2]")
val sc = new SparkContext(conf)
```

#### > 词频统计案例

安装完成后可以先做一个简单的词频统计例子，感受 spark 的魅力。准备一个词频统计的文件样本 `wc.txt`，内容如下：

```text
hadoop,spark,hadoop
spark,flink,flink,spark
hadoop,hadoop
```

在 scala 交互式命令行中执行如下 Scala 语句：

```java
val file = spark.sparkContext.textFile("file:///usr/app/wc.txt")
val wordCounts = file.flatMap(line => line.split(",")).map((word => (word, 1))).reduceByKey(_ + _)
wordCounts.collect
```

执行过程如下，可以看到已经输出了词频统计的结果：

![img](./images/spark-shell.png)

同时还可以通过 Web UI 查看作业的执行情况，访问端口为 `4040`：

![img](./images/spark-shell-web-ui.png)



#### > Scala开发环境配置

Spark 是基于 Scala 语言进行开发的，分别提供了基于 Scala、Java、Python 语言的 API，如果你想使用 Scala 语言进行开发，则需要搭建 Scala 语言的开发环境。

**前置条件**

Scala 的运行依赖于 JDK，所以需要你本机有安装对应版本的 JDK，最新的 Scala 2.12.x 需要 JDK 1.8+。

**安装Scala插件**

IDEA 默认不支持 Scala 语言的开发，需要通过插件进行扩展。打开 IDEA，依次点击 **File** => **settings**=> **plugins** 选项卡，搜索 Scala 插件 (如下图)。找到插件后进行安装，并重启 IDEA 使得安装生效。

![img](./images/idea-scala-plugin.png)

**创建Scala项目** 

在 IDEA 中依次点击 **File** => **New** => **Project** 选项卡，然后选择创建 `Scala—IDEA` 工程：

![img](./images/idea-newproject-scala.png)

**下载Scala SDK** 

1. 方式一

此时看到 `Scala SDK` 为空，依次点击 `Create` => `Download` ，选择所需的版本后，点击 `OK` 按钮进行下载，下载完成点击 `Finish` 进入工程。

![img](./images/idea-scala-select.png)

2. 方式二

方式一是 Scala 官方安装指南里使用的方式，但下载速度通常比较慢，且这种安装下并没有直接提供 Scala 命令行工具。所以个人推荐到官网下载安装包进行安装，下载地址：https://www.scala-lang.org/download/

这里我的系统是 Windows，下载 msi 版本的安装包后，一直点击下一步进行安装，安装完成后会自动配置好环境变量。

![img](./images/scala-other-resources.png)



由于安装时已经自动配置好环境变量，所以 IDEA 会自动选择对应版本的 SDK。

![img](./images/idea-scala-2.1.8.png)

**创建Hello World** 

在工程 `src` 目录上右击 **New** => **Scala class** 创建 `Hello.scala`。输入代码如下，完成后点击运行按钮，成功运行则代表搭建成功。

![img](./images/scala-hello-world.png)

**切换Scala版本** 

在日常的开发中，由于对应软件（如 Spark）的版本切换，可能导致需要切换 Scala 的版本，则可以在 `Project Structures` 中的 `Global Libraries` 选项卡中进行切换。

![img](./images/idea-scala-change.png)

**可能出现的问题** 

在 IDEA 中有时候重新打开项目后，右击并不会出现新建 `scala` 文件的选项，或者在编写时没有 Scala 语法提示，此时可以先删除 `Global Libraries` 中配置好的 SDK，之后再重新添加：

![img](./images/scala-sdk.png)



**另外在 IDEA 中以本地模式运行 Spark 项目是不需要在本机搭建 Spark 和 Hadoop 环境的。**



### （4）Spark部署模式与作业提交 

- **Local  模式**

  > 所谓的 Local 模式，就是不需要其他任何节点资源就可以在本地执行 Spark 代码的环境，一般用于教学，调试，演示等.

- **Standalone  模式**

  > local 本地模式毕竟只是用来进行练习演示的，真实工作中还是要将应用提交到对应的集群中去执行，这里我们来看看只使用 Spark 自身节点运行的集群模式，也就是我们所谓的独立部署（Standalone）模式。Spark 的 Standalone 模式体现了经典的 master-slave 模式。

  集群规划：

  ![img](./images/独立模式-集群规划.PNG)

- **Yarn 模式**

  > 独立部署（Standalone）模式由 Spark 自身提供计算资源，无需其他框架提供资源。这种方式降低了和其他第三方资源框架的耦合性，独立性非常强。但是Spark 主要是计算框架，而不是资源调度框架，所以本身提供的资源调度并不是它的强项，所以还是和其他专业的资源调度框架集成会更靠谱一些。在国内工作中，Yarn 使用的非常多。

  **配置**

  在 `spark-env.sh` 中配置 hadoop 的配置目录的位置，可以使用 `YARN_CONF_DIR` 或 `HADOOP_CONF_DIR` 进行指定：

  ```shell
  YARN_CONF_DIR=/usr/app/hadoop-2.6.0-cdh5.15.2/etc/hadoop
  # JDK安装位置
  JAVA_HOME=/usr/java/jdk1.8.0_201
  ```

  **启动**

  必须要保证 Hadoop 已经启动，这里包括 YARN 和 HDFS 都需要启动，因为在计算过程中 Spark 会使用 HDFS 存储临时文件，如果 HDFS 没有启动，则会抛出异常。

  ```shell
  start-yarn.sh
  start-dfs.sh
  ```

  **提交应用**

  ```shell
  #  以client模式提交到yarn集群 
  spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master yarn \
  --deploy-mode client \
  --executor-memory 2G \
  --num-executors 10 \
  /usr/app/spark-2.4.0-bin-hadoop2.6/examples/jars/spark-examples_2.11-2.4.0.jar \
  100

  #  以cluster模式提交到yarn集群 
  spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master yarn \
  --deploy-mode cluster \
  --executor-memory 2G \
  --num-executors 10 \
  /usr/app/spark-2.4.0-bin-hadoop2.6/examples/jars/spark-examples_2.11-2.4.0.jar \
  100
  ```

  ​

  **部署模式对比**： 

  ![img](./images/部署模式对比.PNG)

  **端口号**：

  > ➢  Spark 查看当前 Spark-shell 运行任务情况端口号：4040（计算）
  >
  > ➢  Spark Master 内部通信服务端口号：7077
  >
  > ➢  Standalone 模式下，Spark Master Web 端口号：8080（资源）
  >
  > ➢  Spark 历史服务器端口号：18080
  >
  > ➢  Hadoop YARN 任务运行情况查看端口号：8088



### （5）Spark运行架构

| Term（术语）        | Meaning（含义）                              |
| --------------- | ---------------------------------------- |
| Application     | Spark 应用程序，由集群上的一个 Driver 节点和多个 Executor 节点组成。 |
| Driver program  | 主运用程序，该进程运行应用的 main() 方法并且创建  SparkContext |
| Cluster manager | 集群资源管理器（例如，Standlone Manager，Mesos，YARN） |
| Worker node     | 执行计算任务的工作节点                              |
| Executor        | 位于工作节点上的应用进程，负责执行计算任务并且将输出数据保存到内存或者磁盘中   |
| Task            | 被发送到 Executor 中的工作单元                     |

![img](./images/spark-集群模式.png)

**核心组件**：

- **Driver**

  > Spark 驱动器节点，用于执行 Spark 任务中的 main 方法，负责实际代码的执行工作。Driver 在 Spark 作业执行时主要负责：
  >
  > ➢  将用户程序转化为作业（job）
  >
  > ➢  在 Executor 之间调度任务(task)
  >
  > ➢  跟踪 Executor 的执行情况
  >
  > ➢  通过 UI 展示查询运行情况
  > &emsp;实际上，我们无法准确地描述 Driver 的定义，因为在整个的编程过程中没有看到任何有关Driver 的字眼。所以简单理解，所谓的 Driver 就是驱使整个应用运行起来的程序，也称之为Driver 类。

-  **Executor**

  > Executor 有两个核心功能：
  >
  > ➢  负责运行组成 Spark 应用的任务，并将结果返回给驱动器进程
  >
  > ➢  它们通过自身的块管理器（Block Manager）为用户程序中要求缓存的 RDD 提供内存式存储。RDD 是直接缓存在 Executor 进程内的，因此任务可以在运行时充分利用缓存数据加速运算

-  **Master & Worker**

  > Spark 集群的独立部署环境中，不需要依赖其他的资源调度框架，自身就实现了资源调度的功能，所以环境中还有其他两个核心组件：Master 和 Worker，这里的 Master 是一个进程，主要负责资源的调度和分配，并进行集群的监控等职责，类似于 Yarn 环境中的 RM, 而Worker 呢，也是进程，一个 Worker 运行在集群中的一台服务器上，由 Master 分配资源对数据进行并行的处理和计算，类似于 Yarn 环境中 NM。

- **ApplicationMaster**

  > Hadoop 用户向 YARN 集群提交应用程序时,提交程序中应该包含 ApplicationMaster，用于向资源调度器申请执行任务的资源容器 Container，运行用户自己的程序任务 job，监控整个任务的执行，跟踪整个任务的状态，处理任务失败等异常情况。
  >
  > 说的简单点就是，ResourceManager（资源）和 Driver（计算）之间的解耦合靠的就是ApplicationMaster。

### （6）核心概念

-  **Executor 与 Core**

  > Spark Executor 是集群中运行在工作节点（Worker）中的一个 JVM 进程，是整个集群中的专门用于计算的节点。在提交应用中，可以提供参数指定计算节点的个数，以及对应的资源。这里的资源一般指的是工作节点 Executor 的内存大小和使用的虚拟 CPU 核（Core）数量。

应用程序相关启动参数如下:

![img](./images/应用程序相关启动参数.PNG)

- **并行度（Parallelism)**

> 在分布式计算框架中一般都是多个任务同时执行，由于任务分布在不同的计算节点进行计算，所以能够真正地实现多任务并行执行，记住，这里是并行，而不是并发。这里我们将整个集群并行执行任务的数量称之为并行度。那么一个作业到底并行度是多少呢？这个取决于框架的默认配置。应用程序也可以在运行过程中动态修改。

### （7）提交流程

&emsp;所谓的提交流程，其实就是我们开发人员根据需求写的应用程序通过 Spark 客户端提交给 Spark 运行环境执行计算的流程。在不同的部署环境中，这个提交过程基本相同，但是又有细微的区别，我们这里不进行详细的比较，但是因为国内工作中，将 Spark 引用部署到Yarn 环境中会更多一些，所以本课程中的提交流程是基于 Yarn 环境的。

![img](./images/提交流程.PNG)

&emsp;Spark 应用程序提交到 Yarn 环境中执行的时候，一般会有两种部署执行的方式：Client和 Cluster。两种模式主要区别在于：Driver 程序的运行节点位置。

**Yarn Client  模式**

&emsp;Client 模式将用于监控和调度的 Driver 模块在客户端执行，而不是在 Yarn 中，所以一般用于测试。

➢  Driver 在任务提交的本地机器上运行。

➢  Driver 启动后会和 ResourceManager 通讯申请启动 ApplicationMaster。

➢  ResourceManager 分配 container，在合适的 NodeManager 上启动 ApplicationMaster，负责向 ResourceManager 申请 Executor 内存。

➢  ResourceManager 接到 ApplicationMaster 的资源申请后会分配 container，然后ApplicationMaster 在资源分配指定的 NodeManager 上启动 Executor 进程。

➢  Executor 进程启动后会向 Driver 反向注册，Executor 全部注册完成后 Driver 开始执行main 函数。

➢  之后执行到 Action 算子时，触发一个 Job，并根据宽依赖开始划分 stage，每个 stage 生成对应的 TaskSet，之后将 task 分发到各个 Executor 上执行。

**Yarn Cluster  模式**

&emsp;Cluster 模式将用于监控和调度的 Driver 模块启动在 Yarn 集群资源中执行。一般应用于实际生产环境。

➢  在 YARN Cluster 模式下，任务提交后会和 ResourceManager 通讯申请启动ApplicationMaster。

➢  随后 ResourceManager 分配 container，在合适的 NodeManager 上启动 ApplicationMaster，此时的 ApplicationMaster 就是 Driver。

➢  Driver 启动后向 ResourceManager 申请 Executor 内存，ResourceManager 接到ApplicationMaster 的资源申请后会分配 container，然后在合适的 NodeManager 上启动Executor 进程。

➢  Executor 进程启动后会向 Driver 反向注册，Executor 全部注册完成后 Driver 开始执行main 函数。

➢  之后执行到 Action 算子时，触发一个 Job，并根据宽依赖开始划分 stage，每个 stage 生成对应的 TaskSet，之后将 task 分发到各个 Executor 上执行。

```shell
nohup spark-submit 
--master [yarn, local[*], mesos]  # 集群的 Master Url
--deploy-mode [client, cluster] # 部署模式
--queue [xxxx] 	   # 使用的队列
--class [xxxx]     # 应用程序主入口类
--conf <key>=<value> \        # 可选配置  
--driver-memory 50g   # driver端内存
--executor-memory 8g  # 一个executor的内存
--executor-cores 4    # 一个executor的核数
--num-executors 64    # 总共多少个executor
[/opt/work/xxxx.jar] # Jar 包路径 
```

### （8）RDD的Transformation 和 Action 常用算子

- **RDD**

  > RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是 Spark 中最基本的数据处理模型。代码中是一个抽象类，它代表一个弹性的、不可变、可分区、里面的元素可并行计算的集合。
  >
  > ​
  >
  > ➢  弹性
  >
  > &emsp;存储的弹性：内存与磁盘的自动切换；
  >
  > &emsp;容错的弹性：数据丢失可以自动恢复；
  >
  > &emsp;计算的弹性：计算出错重试机制；
  >
  > &emsp;分片的弹性：可根据需要重新分片。
  >
  > ➢  分布式：数据存储在大数据集群不同节点上
  >
  > ➢  数据集：RDD 封装了计算逻辑，并不保存数据
  >
  > ➢  数据抽象：RDD 是一个抽象类，需要子类具体实现
  >
  > ➢  不可变：RDD 封装了计算逻辑，是不可以改变的，想要改变，只能产生新的 RDD，在新的 RDD 里面封装计算逻辑
  >
  > ➢  可分区、并行计算

  - **RDD  转换算子:**

    > **map**: 将处理的数据逐条进行映射转换，这里的转换可以是类型的转换，也可以是值的转换。
    >
    > **mapPartitions**: 将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理是指可以进行任意的处理，哪怕是过滤数据。
    >
    > **mapPartitionsWithIndex**:  将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理是指可以进行任意的处理，哪怕是过滤数据，在处理时同时可以获取当前分区索引。
    >
    > **flatMap**: 将处理的数据进行扁平化后再进行映射处理，所以算子也称之为扁平映射。
    >
    > **groupBy**: 将数据根据指定的规则进行分组, 分区默认不变，但是数据会被打乱重新组合，我们将这样的操作称之为 shuffle。极限情况下，数据可能被分在同一个分区中。
    >
    > **filter**: 将数据根据指定的规则进行筛选过滤，符合规则的数据保留，不符合规则的数据丢弃。当数据进行筛选过滤后，分区不变，但是分区内的数据可能不均衡，生产环境下，可能会出现数据倾斜。
    >
    > **sample**: 根据指定的规则从数据集中抽取数据。
    >
    > **distinct**： 将数据集中重复的数据去重。
    >
    > **coalesce**:  根据数据量缩减分区，用于大数据集过滤后，提高小数据集的执行效率当 spark 程序中，存在过多的小任务的时候，可以通过 coalesce 方法，收缩合并分区，减少分区的个数，减小任务调度成本。
    >
    > **repartition**： 该操作内部其实执行的是 coalesce 操作，参数 shuffle 的默认值为 true。无论是将分区数多的RDD 转换为分区数少的 RDD，还是将分区数少的 RDD 转换为分区数多的 RDD，repartition操作都可以完成，因为无论如何都会经 shuffle 过程。
    >
    > **sortBy**：  该操作用于排序数据。在排序之前，可以将数据通过 f 函数进行处理，之后按照 f 函数处理的结果进行排序，默认为升序排列。排序后新产生的 RDD 的分区数与原 RDD 的分区数一致。中间存在 shuffle 的过程。
    >
    > **intersection**： 对源 RDD 和参数 RDD 求交集后返回一个新的 RDD。
    >
    > **union**： 对源 RDD 和参数 RDD 求并集后返回一个新的 RDD。
    >
    > **subtract**： 以一个 RDD 元素为主，去除两个 RDD 中重复元素，将其他元素保留下来。求差集。
    >
    > **zip**：  将两个 RDD 中的元素，以键值对的形式进行合并。其中，键值对中的 Key 为第 1 个 RDD中的元素，Value 为第 2 个 RDD 中的相同位置的元素。
    >
    > **partitionBy**： 将数据按照指定 Partitioner 重新进行分区。Spark 默认的分区器是 HashPartitioner。
    >
    > **reduceByKey**： 可以将数据按照相同的 Key 对 Value 进行聚合。
    >
    > **groupByKey**： 将数据源的数据根据 key 对 value 进行分组。
    >
    > ​
    >
    > (
    >
    > &emsp;从 shuffle  的角度：reduceByKey 和 groupByKey 都存在 shuffle 的操作，但是 reduceByKey可以在 shuffle 前对分区内相同 key 的数据进行预聚合（combine）功能，这样会减少落盘的数据量，而 groupByKey 只是进行分组，不存在数据量减少的问题，reduceByKey 性能比较高。
    >
    > &emsp;从功能的角度：reduceByKey 其实包含分组和聚合的功能。GroupByKey 只能分组，不能聚合，所以在分组聚合的场合下，推荐使用 reduceByKey，如果仅仅是分组而不需要聚合。那么还是只能使用 groupByKey。
    >
    > ）
    >
    > ​
    >
    > **aggregateByKey**:  将数据根据不同的规则进行分区内计算和分区间计算。
    >
    > **foldByKey**：   当分区内计算规则和分区间计算规则相同时，aggregateByKey 就可以简化为 foldByKey。
    >
    > **combineByKey** ：  最通用的对 key-value 型 rdd 进行聚集操作的聚集函数（aggregation function）。类似于aggregate()，combineByKey()允许用户返回值的类型与输入不一致。
    >
    > **sortByKey**： 在一个(K,V)的 RDD 上调用，K 必须实现 Ordered 接口(特质)，返回一个按照 key 进行排序的。
    >
    > **join**： 在类型为(K,V)和(K,W)的 RDD 上调用，返回一个相同 key 对应的所有元素连接在一起的(K,(V,W))的 RDD。
    >
    > **leftOuterJoin**： 类似于 SQL 语句的左外连接。
    >
    > **cogroup**：在类型为(K,V)和(K,W)的 RDD 上调用，返回一个(K,(Iterable<V>,Iterable<W>))类型的 RDD。
    >
    > ​

  - **RDD  行动算子:**

    > **reduce**： 聚集 RDD 中的所有元素，先聚合分区内数据，再聚合分区间数据。
    > **collect**： 在驱动程序中，以数组 Array 的形式返回数据集的所有元素。
    >
    > **count**： 返回 RDD 中元素的个数。
    >
    > **first**： 返回 RDD 中的第一个元素。
    >
    > **take**：返回一个由 RDD 的前 n 个元素组成的数组。
    >
    > **takeOrdered**： 返回该 RDD 排序后的前 n 个元素组成的数组。
    >
    > **aggregate**： 分区的数据通过初始值和分区内的数据进行聚合，然后再和初始值进行分区间的数据聚合。
    >
    > **fold**： 折叠操作，aggregate 的简化版操作。
    >
    > **countByKey**：统计每种 key 的个数。
    >
    > **saveAsTextFile**： 保存。
    >
    > **saveAsObjectFile**： 保存。
    >
    > **saveAsSequenceFile**： 保存。
    >
    > **foreach**：分布式遍历 RDD 中的每一个元素，调用指定函数。

  - **RDD窄依赖:**

    > 窄依赖表示每一个父(上游)RDD 的 Partition 最多被子（下游）RDD 的一个 Partition 使用，窄依赖我们形象的比喻为独生子女。

  - **RDD宽依赖**：

    > 宽依赖表示同一个父（上游）RDD 的 Partition 被多个子（下游）RDD 的 Partition 依赖，会引起 Shuffle，总结：宽依赖我们形象的比喻为多生。

  - **RDD 持久化：**

    > RDD 通过 Cache 或者 Persist 方法将前面的计算结果缓存，默认情况下会把数据以缓存在 JVM 的堆内存中。但是并不是这两个方法被调用时立即缓存，而是触发后面的 action 算子时，该 RDD 将会被缓存在计算节点的内存中，并供后面重用。
    >
    > 缓存有可能丢失，或者存储于内存的数据由于内存不足而被删除，RDD 的缓存容错机制保证了即使缓存丢失也能保证计算的正确执行。通过基于 RDD 的一系列转换，丢失的数据会被重算，由于 RDD 的各个 Partition 是相对独立的，因此只需要计算丢失的部分即可，并不需要重算全部 Partition。
    >
    > Spark 会自动对一些 Shuffle 操作的中间数据做持久化操作(比如：reduceByKey)。这样做的目的是为了当一个节点 Shuffle 失败了避免重新计算整个输入。但是，在实际使用的时候，如果想重用数据，仍然建议调用 persist 或 cache。

  - **RDD CheckPoint  检查点：**

    > 所谓的检查点其实就是通过将 RDD 中间结果写入磁盘由于血缘依赖过长会造成容错成本过高，这样就不如在中间阶段做检查点容错，如果检查点之后有节点出现问题，可以从检查点开始重做血缘，减少了开销。对 RDD 进行 checkpoint 操作并不会马上被执行，必须执行 Action 操作才能触发。

  - **缓存和检查点区别：**

    > 1）Cache 缓存只是将数据保存起来，不切断血缘依赖。Checkpoint 检查点切断血缘依赖。
    >
    > 2）Cache 缓存的数据通常存储在磁盘、内存等地方，可靠性低。Checkpoint 的数据通常存储在 HDFS 等容错、高可用的文件系统，可靠性高。
    >
    > 3）建议对 checkpoint()的 RDD 使用 Cache 缓存，这样 checkpoint 的 job 只需从 Cache 缓存中读取数据即可，否则需要再从头计算一次 RDD。

  - **RDD  分区器：**

    > Spark 目前支持 Hash 分区和 Range 分区，和用户自定义分区。Hash 分区为当前的默认分区。分区器直接决定了 RDD 中分区的个数、RDD 中每条数据经过 Shuffle 后进入哪个分区，进而决定了Reduce 的个数。

<font color='red'>从计算的角度, 算子以外的代码都是在 Driver 端执行, 算子里面的代码都是在 Executor
端执行。</font>

#### > Transformation

spark 常用的 Transformation 算子如下表：

| Transformation 算子                        | Meaning（含义）                              |
| ---------------------------------------- | ---------------------------------------- |
| **map**(*func*)                          | 对原 RDD 中每个元素运用 *func* 函数，并生成新的 RDD       |
| **filter**(*func*)                       | 对原 RDD 中每个元素使用*func* 函数进行过滤，并生成新的 RDD    |
| **flatMap**(*func*)                      | 与 map 类似，但是每一个输入的 item 被映射成 0 个或多个输出的 items（ *func* 返回类型需要为 Seq ）。 |
| **mapPartitions**(*func*)                | 与 map 类似，但函数单独在 RDD 的每个分区上运行， *func*函数的类型为  Iterator\<T> => Iterator\<U> ，其中 T 是 RDD 的类型，即 RDD[T] |
| **mapPartitionsWithIndex**(*func*)       | 与 mapPartitions 类似，但 *func* 类型为 (Int, Iterator\<T>) => Iterator\<U> ，其中第一个参数为分区索引 |
| **sample**(*withReplacement*, *fraction*, *seed*) | 数据采样，有三个可选参数：设置是否放回（withReplacement）、采样的百分比（*fraction*）、随机数生成器的种子（seed）； |
| **union**(*otherDataset*)                | 合并两个 RDD                                 |
| **intersection**(*otherDataset*)         | 求两个 RDD 的交集                              |
| **distinct**([*numTasks*]))              | 去重                                       |
| **groupByKey**([*numTasks*])             | 按照 key 值进行分区，即在一个 (K, V) 对的 dataset 上调用时，返回一个 (K, Iterable\<V>) <br/>**Note:** 如果分组是为了在每一个 key 上执行聚合操作（例如，sum 或 average)，此时使用 `reduceByKey` 或 `aggregateByKey` 性能会更好<br>**Note:** 默认情况下，并行度取决于父 RDD 的分区数。可以传入 `numTasks` 参数进行修改。 |
| **reduceByKey**(*func*, [*numTasks*])    | 按照 key 值进行分组，并对分组后的数据执行归约操作。             |
| **aggregateByKey**(*zeroValue*,*numPartitions*)(*seqOp*, *combOp*, [*numTasks*]) | 当调用（K，V）对的数据集时，返回（K，U）对的数据集，其中使用给定的组合函数和 zeroValue 聚合每个键的值。与 groupByKey 类似，reduce 任务的数量可通过第二个参数进行配置。 |
| **sortByKey**([*ascending*], [*numTasks*]) | 按照 key 进行排序，其中的 key 需要实现 Ordered 特质，即可比较 |
| **join**(*otherDataset*, [*numTasks*])   | 在一个 (K, V) 和 (K, W) 类型的 dataset 上调用时，返回一个 (K, (V, W)) pairs 的 dataset，等价于内连接操作。如果想要执行外连接，可以使用 `leftOuterJoin`, `rightOuterJoin` 和 `fullOuterJoin` 等算子。 |
| **cogroup**(*otherDataset*, [*numTasks*]) | 在一个 (K, V) 对的 dataset 上调用时，返回一个 (K, (Iterable\<V>, Iterable\<W>)) tuples 的 dataset。 |
| **cartesian**(*otherDataset*)            | 在一个 T 和 U 类型的 dataset 上调用时，返回一个 (T, U) 类型的 dataset（即笛卡尔积）。 |
| **coalesce**(*numPartitions*)            | 将 RDD 中的分区数减少为 numPartitions。            |
| **repartition**(*numPartitions*)         | 随机重新调整 RDD 中的数据以创建更多或更少的分区，并在它们之间进行平衡。   |
| **repartitionAndSortWithinPartitions**(*partitioner*) | 根据给定的 partitioner（分区器）对 RDD 进行重新分区，并对分区中的数据按照 key 值进行排序。这比调用 `repartition` 然后再 sorting（排序）效率更高，因为它可以将排序过程推送到 shuffle 操作所在的机器。 |

下面分别给出这些算子的基本使用示例：

**1.1 map** 

```java
val list = List(1,2,3)
sc.parallelize(list).map(_ * 10).foreach(println)

// 输出结果： 10 20 30 （这里为了节省篇幅去掉了换行,后文亦同）
```

**1.2 filter**  

```java
val list = List(3, 6, 9, 10, 12, 21)
sc.parallelize(list).filter(_ >= 10).foreach(println)

// 输出： 10 12 21
```

**1.3 flatMap** 

`flatMap(func)` 与 `map` 类似，但每一个输入的 item 会被映射成 0 个或多个输出的 items（ *func* 返回类型需要为 `Seq`）。

```java
val list = List(List(1, 2), List(3), List(), List(4, 5))
sc.parallelize(list).flatMap(_.toList).map(_ * 10).foreach(println)

// 输出结果 ： 10 20 30 40 50
```

flatMap 这个算子在日志分析中使用概率非常高，这里进行一下演示：拆分输入的每行数据为单个单词，并赋值为 1，代表出现一次，之后按照单词分组并统计其出现总次数，代码如下：

```java
val lines = List("spark flume spark",
                 "hadoop flume hive")
sc.parallelize(lines).flatMap(line => line.split(" ")).
map(word=>(word,1)).reduceByKey(_+_).foreach(println)

// 输出：
(spark,2)
(hive,1)
(hadoop,1)
(flume,2)
```

**1.4 mapPartitions** 

与 map 类似，但函数单独在 RDD 的每个分区上运行， *func*函数的类型为 `Iterator<T> => Iterator<U>` (其中 T 是 RDD 的类型)，即输入和输出都必须是可迭代类型。

```java
val list = List(1, 2, 3, 4, 5, 6)
sc.parallelize(list, 3).mapPartitions(iterator => {
  val buffer = new ListBuffer[Int]
  while (iterator.hasNext) {
    buffer.append(iterator.next() * 100)
  }
  buffer.toIterator
}).foreach(println)
//输出结果
100 200 300 400 500 600
```

**1.5 mapPartitionsWithIndex**

  与 mapPartitions 类似，但 *func* 类型为 `(Int, Iterator<T>) => Iterator<U>` ，其中第一个参数为分区索引。

```java
val list = List(1, 2, 3, 4, 5, 6)
sc.parallelize(list, 3).mapPartitionsWithIndex((index, iterator) => {
  val buffer = new ListBuffer[String]
  while (iterator.hasNext) {
    buffer.append(index + "分区:" + iterator.next() * 100)
  }
  buffer.toIterator
}).foreach(println)
//输出
0 分区:100
0 分区:200
1 分区:300
1 分区:400
2 分区:500
2 分区:600
```

**1.6 sample**

  数据采样。有三个可选参数：设置是否放回 (withReplacement)、采样的百分比 (fraction)、随机数生成器的种子 (seed) ：

```java
val list = List(1, 2, 3, 4, 5, 6)
sc.parallelize(list).sample(withReplacement = false, fraction = 0.5).foreach(println)
```

**1.7 union**

合并两个 RDD：

```java
val list1 = List(1, 2, 3)
val list2 = List(4, 5, 6)
sc.parallelize(list1).union(sc.parallelize(list2)).foreach(println)
// 输出: 1 2 3 4 5 6
```

**1.8 intersection**

求两个 RDD 的交集：

```java
val list1 = List(1, 2, 3, 4, 5)
val list2 = List(4, 5, 6)
sc.parallelize(list1).intersection(sc.parallelize(list2)).foreach(println)
// 输出:  4 5
```

**1.9 distinct**

去重：

```java
val list = List(1, 2, 2, 4, 4)
sc.parallelize(list).distinct().foreach(println)
// 输出: 4 1 2
```

**1.10 groupByKey**

按照键进行分组：

```java
val list = List(("hadoop", 2), ("spark", 3), ("spark", 5), ("storm", 6), ("hadoop", 2))
sc.parallelize(list).groupByKey().map(x => (x._1, x._2.toList)).foreach(println)

//输出：
(spark,List(3, 5))
(hadoop,List(2, 2))
(storm,List(6))
```

**1.11 reduceByKey**

按照键进行归约操作：

```java
val list = List(("hadoop", 2), ("spark", 3), ("spark", 5), ("storm", 6), ("hadoop", 2))
sc.parallelize(list).reduceByKey(_ + _).foreach(println)

//输出
(spark,8)
(hadoop,4)
(storm,6)
```

**1.12 sortBy & sortByKey**

按照键进行排序：

```java
val list01 = List((100, "hadoop"), (90, "spark"), (120, "storm"))
sc.parallelize(list01).sortByKey(ascending = false).foreach(println)
// 输出
(120,storm)
(90,spark)
(100,hadoop)
```

按照指定元素进行排序：

```java
val list02 = List(("hadoop",100), ("spark",90), ("storm",120))
sc.parallelize(list02).sortBy(x=>x._2,ascending=false).foreach(println)
// 输出
(storm,120)
(hadoop,100)
(spark,90)
```

**1.13 join**

在一个 (K, V) 和 (K, W) 类型的 Dataset 上调用时，返回一个 (K, (V, W)) 的 Dataset，等价于内连接操作。如果想要执行外连接，可以使用 `leftOuterJoin`, `rightOuterJoin` 和 `fullOuterJoin` 等算子。

```java
val list01 = List((1, "student01"), (2, "student02"), (3, "student03"))
val list02 = List((1, "teacher01"), (2, "teacher02"), (3, "teacher03"))
sc.parallelize(list01).join(sc.parallelize(list02)).foreach(println)

// 输出
(1,(student01,teacher01))
(3,(student03,teacher03))
(2,(student02,teacher02))
```

**1.14 cogroup**

在一个 (K, V) 对的 Dataset 上调用时，返回多个类型为 (K, (Iterable\<V>, Iterable\<W>)) 的元组所组成的 Dataset。

```java
val list01 = List((1, "a"),(1, "a"), (2, "b"), (3, "e"))
val list02 = List((1, "A"), (2, "B"), (3, "E"))
val list03 = List((1, "[ab]"), (2, "[bB]"), (3, "eE"),(3, "eE"))
sc.parallelize(list01).cogroup(sc.parallelize(list02),sc.parallelize(list03)).foreach(println)

// 输出： 同一个 RDD 中的元素先按照 key 进行分组，然后再对不同 RDD 中的元素按照 key 进行分组
(1,(CompactBuffer(a, a),CompactBuffer(A),CompactBuffer([ab])))
(3,(CompactBuffer(e),CompactBuffer(E),CompactBuffer(eE, eE)))
(2,(CompactBuffer(b),CompactBuffer(B),CompactBuffer([bB])))

```

**1.15 cartesian**

计算笛卡尔积：

```java
val list1 = List("A", "B", "C")
val list2 = List(1, 2, 3)
sc.parallelize(list1).cartesian(sc.parallelize(list2)).foreach(println)

//输出笛卡尔积
(A,1)
(A,2)
(A,3)
(B,1)
(B,2)
(B,3)
(C,1)
(C,2)
(C,3)
```

**1.16 aggregateByKey**

当调用（K，V）对的数据集时，返回（K，U）对的数据集，其中使用给定的组合函数和 zeroValue 聚合每个键的值。与 `groupByKey` 类似，reduce 任务的数量可通过第二个参数 `numPartitions` 进行配置。示例如下：

```java
// 为了清晰，以下所有参数均使用具名传参
val list = List(("hadoop", 3), ("hadoop", 2), ("spark", 4), ("spark", 3), ("storm", 6), ("storm", 8))
sc.parallelize(list,numSlices = 2).aggregateByKey(zeroValue = 0,numPartitions = 3)(
      seqOp = math.max(_, _),
      combOp = _ + _
    ).collect.foreach(println)
//输出结果：
(hadoop,3)
(storm,8)
(spark,7)
```

这里使用了 `numSlices = 2` 指定 aggregateByKey 父操作 parallelize 的分区数量为 2，其执行流程如下：

![img](./images/spark-aggregateByKey.png)

基于同样的执行流程，如果 `numSlices = 1`，则意味着只有输入一个分区，则其最后一步 combOp 相当于是无效的，执行结果为：

```properties
(hadoop,3)
(storm,8)
(spark,4)
```

同样的，如果每个单词对一个分区，即 `numSlices = 6`，此时相当于求和操作，执行结果为：

```properties
(hadoop,5)
(storm,14)
(spark,7)
```

`aggregateByKey(zeroValue = 0,numPartitions = 3)` 的第二个参数 `numPartitions` 决定的是输出 RDD 的分区数量，想要验证这个问题，可以对上面代码进行改写，使用 `getNumPartitions` 方法获取分区数量：

```java
sc.parallelize(list,numSlices = 6).aggregateByKey(zeroValue = 0,numPartitions = 3)(
  seqOp = math.max(_, _),
  combOp = _ + _
).getNumPartitions
```

![img](./images/spark-getpartnum.png) 

#### > Action

Spark 常用的 Action 算子如下：

| Action（动作）                               | Meaning（含义）                              |
| ---------------------------------------- | ---------------------------------------- |
| **reduce**(*func*)                       | 使用函数*func*执行归约操作                         |
| **collect**()                            | 以一个 array 数组的形式返回 dataset 的所有元素，适用于小结果集。 |
| **count**()                              | 返回 dataset 中元素的个数。                       |
| **first**()                              | 返回 dataset 中的第一个元素，等价于 take(1)。          |
| **take**(*n*)                            | 将数据集中的前 *n* 个元素作为一个 array 数组返回。          |
| **takeSample**(*withReplacement*, *num*, [*seed*]) | 对一个 dataset 进行随机抽样                       |
| **takeOrdered**(*n*, *[ordering]*)       | 按自然顺序（natural order）或自定义比较器（custom comparator）排序后返回前 *n* 个元素。只适用于小结果集，因为所有数据都会被加载到驱动程序的内存中进行排序。 |
| **saveAsTextFile**(*path*)               | 将 dataset 中的元素以文本文件的形式写入本地文件系统、HDFS 或其它 Hadoop 支持的文件系统中。Spark 将对每个元素调用 toString 方法，将元素转换为文本文件中的一行记录。 |
| **saveAsSequenceFile**(*path*)           | 将 dataset 中的元素以 Hadoop SequenceFile 的形式写入到本地文件系统、HDFS 或其它 Hadoop 支持的文件系统中。该操作要求 RDD 中的元素需要实现 Hadoop 的 Writable 接口。对于 Scala 语言而言，它可以将 Spark 中的基本数据类型自动隐式转换为对应 Writable 类型。(目前仅支持 Java and Scala) |
| **saveAsObjectFile**(*path*)             | 使用 Java 序列化后存储，可以使用 `SparkContext.objectFile()` 进行加载。(目前仅支持 Java and Scala) |
| **countByKey**()                         | 计算每个键出现的次数。                              |
| **foreach**(*func*)                      | 遍历 RDD 中每个元素，并对其执行*fun*函数                |

**2.1 reduce**

使用函数*func*执行归约操作：

```java
 val list = List(1, 2, 3, 4, 5)
sc.parallelize(list).reduce((x, y) => x + y)
sc.parallelize(list).reduce(_ + _)

// 输出 15
```

**2.2 takeOrdered**

按自然顺序（natural order）或自定义比较器（custom comparator）排序后返回前 *n* 个元素。需要注意的是 `takeOrdered` 使用隐式参数进行隐式转换，以下为其源码。所以在使用自定义排序时，需要继承 `Ordering[T]` 实现自定义比较器，然后将其作为隐式参数引入。

```java
def takeOrdered(num: Int)(implicit ord: Ordering[T]): Array[T] = withScope {
  .........
}
```

自定义规则排序：

```java
// 继承 Ordering[T],实现自定义比较器，按照 value 值的长度进行排序
class CustomOrdering extends Ordering[(Int, String)] {
    override def compare(x: (Int, String), y: (Int, String)): Int
    = if (x._2.length > y._2.length) 1 else -1
}

val list = List((1, "hadoop"), (1, "storm"), (1, "azkaban"), (1, "hive"))
//  引入隐式默认值
implicit val implicitOrdering = new CustomOrdering
sc.parallelize(list).takeOrdered(5)

// 输出： Array((1,hive), (1,storm), (1,hadoop), (1,azkaban)
```

**2.3 countByKey**

计算每个键出现的次数：

```java
val list = List(("hadoop", 10), ("hadoop", 10), ("storm", 3), ("storm", 3), ("azkaban", 1))
sc.parallelize(list).countByKey()

// 输出： Map(hadoop -> 2, storm -> 2, azkaban -> 1)
```

**2.4 saveAsTextFile**

将 dataset 中的元素以文本文件的形式写入本地文件系统、HDFS 或其它 Hadoop 支持的文件系统中。Spark 将对每个元素调用 toString 方法，将元素转换为文本文件中的一行记录。

```java
val list = List(("hadoop", 10), ("hadoop", 10), ("storm", 3), ("storm", 3), ("azkaban", 1))
sc.parallelize(list).saveAsTextFile("/usr/file/temp")
```

参考资料

[RDD Programming Guide](http://spark.apache.org/docs/latest/rdd-programming-guide.html#rdd-programming-guide)



### （9）累加器

➢  累加器：分布式共享<font color='red'>只写</font>变量

> 累加器用来把 Executor 端变量信息聚合到 Driver 端。在 Driver 程序中定义的变量，在Executor 端的每个 Task 都会得到这个变量的一份新的副本，每个 task 更新这些副本的值后，传回 Driver 端进行merge。

```java
val rdd = sc.makeRDD(List(1,2,3,4,5))
// 声明累加器
var sum = sc.longAccumulator("sum");
rdd.foreach(
num => {
// 使用累加器
sum.add(num)
}
)
// 获取累加器的值
println("sum = " + sum.value)
```

<font color='red'>**理解闭包**</font>

**1. Scala 中闭包的概念**

这里先介绍一下 Scala 中关于闭包的概念：

```java
var more = 10
val addMore = (x: Int) => x + more
```

如上函数 `addMore` 中有两个变量 x 和 more:

- **x** : 是一个绑定变量 (bound variable)，因为其是该函数的入参，在函数的上下文中有明确的定义；
- **more** : 是一个自由变量 (free variable)，因为函数字面量本生并没有给 more 赋予任何含义。

按照定义：在创建函数时，如果需要捕获自由变量，那么包含指向被捕获变量的引用的函数就被称为闭包函数。

**2. Spark 中的闭包**

在实际计算时，Spark 会将对 RDD 操作分解为 Task，Task 运行在 Worker Node 上。在执行之前，Spark 会对任务进行闭包，如果闭包内涉及到自由变量，则程序会进行拷贝，并将副本变量放在闭包中，之后闭包被序列化并发送给每个执行者。因此，当在 foreach 函数中引用 `counter` 时，它将不再是 Driver 节点上的 `counter`，而是闭包中的副本 `counter`，默认情况下，副本 `counter` 更新后的值不会回传到 Driver，所以 `counter` 的最终值仍然为零。

需要注意的是：在 Local 模式下，有可能执行 `foreach` 的 Worker Node 与 Diver 处在相同的 JVM，并引用相同的原始 `counter`，这时候更新可能是正确的，但是在集群模式下一定不正确。所以在遇到此类问题时应优先使用累加器。

累加器的原理实际上很简单：就是将每个副本变量的最终值传回 Driver，由 Driver 聚合后得到最终值，并更新原始变量。



### （10）广播变量

➢  广播变量：分布式共享<font color='red'>只读</font>变量

> 广播变量用来高效分发较大的对象。向所有工作节点发送一个较大的只读值，以供一个或多个 Spark 操作使用。比如，如果你的应用需要向所有节点发送一个较大的只读查询表，广播变量用起来都很顺手。在多个并行操作中使用同一个变量，但是 Spark 会为每个任务分别发送。

```java
val rdd1 = sc.makeRDD(List( ("a",1), ("b", 2), ("c", 3), ("d", 4) ),4)
val list = List( ("a",4), ("b", 5), ("c", 6), ("d", 7) )
// 声明广播变量
val broadcast: Broadcast[List[(String, Int)]] = sc.broadcast(list)
val resultRDD: RDD[(String, (Int, Int))] = rdd1.map {
case (key, num) => {
var num2 = 0
// 使用广播变量
for ((k, v) <- broadcast.value) {
if (k == key) {
num2 = v
}
}
(key, (num, num2))
}
}
```



## 二、 Spark-SQL

### （1）Spark - SQL 的 DataFrame / DataSet

#### > Spark SQL 的简介

> Spark SQL 是 Spark 用于结构化数据(structured data)处理的 Spark 模块。
>
> Spark SQL 它提供了 2 个编程抽象, 类似 Spark Core 中的  RDD
>
> （1）DataFrame
>
> （2）DataSet

#### > Spark SQL 的特点

> **Integrated( 易整合)**：无缝的整合了 SQL 查询和 Spark 编程。
>
> **Uniform Data Access( 统一的数据访问方式)**：使用相同的方式连接不同的数据源。
>
> **Hive Integration( 集成 Hive)**：在已有的仓库上直接运行 SQL 或者 HiveQL。
>
> **Standard Connectivity( 标准的连接方式)**：通过 JDBC 或者 ODBC 来连接。

#### > 什么是DataFrame

> 1. 与 RDD 类似， DataFrame 也是一个分布式数据容器。
> 2. 然而 DataFrame 更像传统数据库的二维表格，除了数据以外，还记录数据的结构信息，即 schema 。同时，与 Hive 类似， DataFrame 也支持嵌套数据类型（ struct 、 array 和 map ）。
> 3. 从 API 易用性的角度上看， DataFrame API 提供的是一套高层的关系操作，比函数式的RDD API 要更加友好，门槛更低。
> 4. 性能上比  RDD 要高，主要原因： 优化的执行计划：查询计划通过 Spark catalyst optimiser进行优化。

#### > 什么是DataSet

> 1. 是 DataFrame API 的一个扩展，是 SparkSQL 最新的数据抽象(1.6 新增)。
> 2. 用户友好的 API 风格，既具有类型安全检查也具有 DataFrame 的查询优化特性。
> 3. Dataset 支持编解码器，当需要访问非堆上的数据时可以避免反序列化整个对象，提高了效率。
> 4. 样例类被用来在 DataSet 中定义数据的结构信息，样例类中每个属性的名称直接映射到 DataSet 中的字段名称。
> 5. DataFrame 是 DataSet 的特列， DataFrame=DataSet[Row] ，所以可以通过as 方法将 DataFrame 转换为 DataSet 。 Row 是一个类型，跟 Car 、 Person 这些的类型一样，所有的表结构信息都用 Row 来表示。
> 6. DataSet 是强类型的。比如可以有 DataSet[Car] ， DataSet[Person] .
> 7. DataFrame 只是知道字段，但是不知道字段的类型，所以在执行这些操作的时候是没办法在编译的时候检查是否类型失败的，比如你可以对一个String 进行减法操作，在执行的时候才报错，而 DataSet 不仅仅知道字段，而且知道字段类型，所以有更严格的错误检查。就跟 JSON 对象和类对象之间的类比。

### （2）Spark - SQL 核心编程

>  SparkSession 是 Spark 最新的 SQL 查询起始点，实质上是 SQLContext和 HiveContext 的组合。
>
>  我们使用 spark-shell 的时候, spark 会自动的创建一个叫做 spark 的 SparkSession ,就像我们以前可以自动获取到一个 sc 来表示 SparkContext

#### > RDD、DataFrame、DataSet的关系 / 转化

> - RDDs 适合非结构化数据的处理，而 DataFrame & DataSet 更适合结构化数据和半结构化的处理；
>
> - DataFrame & DataSet 可以通过统一的 Structured API 进行访问，而 RDDs 则更适合函数式编程的场景；
>
> - 相比于 DataFrame 而言，DataSet 是强类型的 (Typed)，有着更为严格的静态类型检查；
>
> - DataSets、DataFrames、SQL 的底层都依赖了 RDDs API，并对外提供结构化的访问接口。
>
>   ​
>
> - DataSets、DataFrames、RDD区别：
>
>   - **DataSets** 
>     1. Dataset 和 DataFrame 拥有完全相同的成员函数，区别只是每一行的数据类型不同 ，DataFrame 其实就是 DataSet 的一个特例。
>     2. DataFrame 也可以叫 Dataset[Row] ,每一行的类型是 Row ，不解析，每一行究竟有哪些字段，各个字段又是什么类型都无从得知，只能用上面提到的getAS 方法或者共性中的第七条提到的模式匹配拿出特定字段。而Dataset 中，每一行是什么类型是不一定的，在自定义了 case class 之后可以很自由的获得每一行的信息
>   - **RDD**：
>     1. RDD 一般和 spark mlib 同时使用。
>     2. RDD 不支持 sparksql 操作。
>
>   - **DataFrames** 
>     1.  与 RDD 和 Dataset 不同， DataFrame 每一行的类型固定为 Row ，每一列的值没法直接访问，只有通过解析才能获取各个字段的值。
>     2.  DataFrame 与 DataSet 一般不与 spark mlib 同时使用。
>     3.  DataFrame 与 DataSet 均支持 SparkSQL 的操作，比如 select ， groupby之类，还能注册临时表/视窗，进行 sql 语句操作。
>     4.  DataFrame 与 DataSet 支持一些特别方便的保存方式，比如保存成 csv ，可以带上表头，这样每一列的字段名一目了然。

![img](./images/互换.PNG)



  #### > 使用 IDEA  创建 SparkSQL  程序

**步骤 1** :  添加 SparkSQL  依赖

```properties
<dependency>
  <groupId>org.apache.spark</groupId>
  <artifactId>spark-sql_2.11</artifactId>
  <version>2.1.1</version>
</dependency>
```

**步骤 2** :  具体代码

```java
object DataFrameDemo {
	def main(args: Array[String]): Unit = {
		// 创建一个新的 SparkSession 对象
		val spark: SparkSession = SparkSession.builder()
			.master("local[*]")
			.appName("Word Count")
			.getOrCreate()
          
			// 导入用到隐式转换. 如果想要使用: $"age" 则必须导入
            import spark.implicits._
          
			val df = spark.read.json("file://" +
				ClassLoader.getSystemResource("user.json").getPath)
            // 打印信息
            df.show
            // 查找年龄大于 19 岁的
            df.filter($"age" > 19).show
          
            // 创建临时表
            df.createTempView("user")
            spark.sql("select * from user where age > 19").show
          
            //关闭连接
            spark.stop()
	}
}
```

#### > **Columns列操作**

引用列

Spark 支持多种方法来构造和引用列，最简单的是使用 `col() ` 或 `column() ` 函数。

```java
col("colName")
column("colName")

// 对于 Scala 语言而言，还可以使用$"myColumn"和'myColumn 这两种语法糖进行引用。
df.select($"ename", $"job").show()
df.select('ename, 'job).show()
```

新增列

```java
// 基于已有列值新增列
df.withColumn("upSal",$"sal"+1000)
// 基于固定值新增列
df.withColumn("intCol",lit(1000))
```

删除列

```java
// 支持删除多个列
df.drop("comm","job").show()
```

重命名列

```java
df.withColumnRenamed("comm", "common").show()
```

需要说明的是新增，删除，重命名列都会产生新的 DataFrame，原来的 DataFrame 不会被改变。

<br/>

#### > 自定义 Spark-SQL函数

**步骤1**： 写一个生成md5的函数

```java
import java.security.MessageDigest
import java.math.BigInteger

object tools_functions { 
	def getMD5(str: String): String = {
    // 第一步，获取MessageDigest对象，参数为MD5表示这是一个MD5算法
    val md5 = MessageDigest.getInstance("MD5")
    // 第二步，计算MD5值
    val array = md5.digest(str.getBytes("UTF-8"))
    // 第三步，结果转换并返回
    val bigInt = new BigInteger(1, array)

    val md5Str = bigInt.toString(16)

    md5Str
  }
}
```

**步骤二**： 导入udf

```java
import org.apache.spark.sql.SparkSession
import com.martinhub.utils.tools_functions.getMD5
import org.apache.spark.sql.functions.udf

object data_process {
  // udf注册自定义函数
  private val generateMD5 = udf(getMD5 _) //生成MD5

  def main(args: Array[String]): Unit = {
    // 创建spark session
    val spark: SparkSession = SparkSession
      .builder()
      .appName("Extract Data")
      .enableHiveSupport()
      .getOrCreate()
    spark.sparkContext.setLogLevel("ERROR")
    import spark.implicits._
    
    //读取Hive表
     val res = spark.table(s"sys.data_table")
     .withColumn("md5_id", generateMD5($"tel_num"))
     
     ........
     ........
     ........
```



### （3）Spark - SQL 的数据源

#### > 前情概要

**通用加载和保存函数**

> 1.  spark.read.load 是加载数据的通用方法.
> 2.  df.write.save 是保存数据的通用方法.

**手动指定选项**

> 也可以手动给数据源指定一些额外的选项. 数据源应该用全名称来指定, 但是对一些内置的数据源也可以使用短名称: 
>
> - CSV
> - JSON
> - Parquet
> - ORC
> - JDBC
> - text

- **读数据格式** 

  所有读取 API 遵循以下调用格式：

  ```java
  // 格式
  DataFrameReader.format(...).option("key", "value").schema(...).load()

  // 示例
  spark.read.format("csv")
  .option("mode", "FAILFAST")          // 读取模式
  .option("inferSchema", "true")       // 是否自动推断 schema
  .option("path", "path/to/file(s)")   // 文件路径
  .schema(someSchema)                  // 使用预定义的 schema      
  .load()
  ```

  读取模式有以下三种可选项：

  | 读模式             | 描述                                       |
  | --------------- | ---------------------------------------- |
  | `permissive`    | 当遇到损坏的记录时，将其所有字段设置为 null，并将所有损坏的记录放在名为 _corruption t_record 的字符串列中 |
  | `dropMalformed` | 删除格式不正确的行                                |
  | `failFast`      | 遇到格式不正确的数据时立即失败                          |

- **写数据格式**

  ```java
  // 格式
  DataFrameWriter.format(...).option(...).partitionBy(...).bucketBy(...).sortBy(...).save()

  //示例
  dataframe.write.format("csv")
  .option("mode", "OVERWRITE")         //写模式
  .option("dateFormat", "yyyy-MM-dd")  //日期格式
  .option("path", "path/to/file(s)")
  .save()
  ```

  写数据模式有以下四种可选项：

  | Scala/Java               | 描述                             |
  | :----------------------- | :----------------------------- |
  | `SaveMode.ErrorIfExists` | 如果给定的路径已经存在文件，则抛出异常，这是写数据默认的模式 |
  | `SaveMode.Append`        | 数据以追加的方式写入                     |
  | `SaveMode.Overwrite`     | 数据以覆盖的方式写入                     |
  | `SaveMode.Ignore`        | 如果给定的路径已经存在文件，则不做任何操作          |

**文件保存选项(SaveMode)**

> 保存操作可以使用 SaveMode, 用来指明如何处理数据. 使用 mode() 方法来设置.
>
> 有一点很重要: 这些 SaveMode 都是没有加锁的, 也不是原子操作. 还有, 如果你执行的是Overwrite 操作, 在写入新的数据之前会先删除旧的数据.

| Scala/Java                       | Any Language      | Meaning       |
| -------------------------------- | ----------------- | ------------- |
| SaveMode.ErrorIfExists (default) | "error" (default) | 如果文件已经存在则抛出异常 |
| SaveMode.Append                  | "append"          | 如果文件已经存在则追加   |
| SaveMode.Overwrite               | "overwrite"       | 如果文件已经存在则覆盖   |
| SaveMode.Ignore                  | "ignore"          | 如果文件已经存在则忽略   |

#### > 数据源 - JDBC

> Spark 同样支持与传统的关系型数据库进行数据读写。但是 Spark 程序默认是没有提供数据库驱动的，所以在使用前需要将对应的数据库驱动上传到安装目录下的 `jars` 目录中。下面示例使用的是 Mysql 数据库，使用前需要将对应的 `mysql-connector-java-x.x.x.jar` 上传到 `jars` 目录下。

 **导入依赖:** 

```properties
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.27</version>
</dependency>
```

 **读取数据** 

读取全表数据示例如下，这里的 `help_keyword` 是 mysql 内置的字典表，只有 `help_keyword_id` 和 `name` 两个字段。

```java
spark.read
.format("jdbc")
.option("driver", "com.mysql.jdbc.Driver")            //驱动
.option("url", "jdbc:mysql://127.0.0.1:3306/mysql")   //数据库地址
.option("dbtable", "help_keyword")                    //表名
.option("user", "root").option("password","root").load().show(10)
```

从查询结果读取数据：

```java
val pushDownQuery = """(SELECT * FROM help_keyword WHERE help_keyword_id <20) AS help_keywords"""
spark.read.format("jdbc")
.option("url", "jdbc:mysql://127.0.0.1:3306/mysql")
.option("driver", "com.mysql.jdbc.Driver")
.option("user", "root").option("password", "root")
.option("dbtable", pushDownQuery)
.load().show()
```

也可以使用如下的写法进行数据的过滤：

```java
val props = new java.util.Properties
props.setProperty("driver", "com.mysql.jdbc.Driver")
props.setProperty("user", "root")
props.setProperty("password", "root")
val predicates = Array("help_keyword_id < 10  OR name = 'WHEN'")   //指定数据过滤条件
spark.read.jdbc("jdbc:mysql://127.0.0.1:3306/mysql", "help_keyword", predicates, props).show() 
```

可以使用 `numPartitions` 指定读取数据的并行度：

```java
option("numPartitions", 10)
```

在这里，除了可以指定分区外，还可以设置上界和下界，任何小于下界的值都会被分配在第一个分区中，任何大于上界的值都会被分配在最后一个分区中。

```java
val colName = "help_keyword_id"   //用于判断上下界的列
val lowerBound = 300L    //下界
val upperBound = 500L    //上界
val numPartitions = 10   //分区综述
val jdbcDf = spark.read.jdbc("jdbc:mysql://127.0.0.1:3306/mysql","help_keyword",
                             colName,lowerBound,upperBound,numPartitions,props)
```

想要验证分区内容，可以使用 `mapPartitionsWithIndex` 这个算子，代码如下：

```java
jdbcDf.rdd.mapPartitionsWithIndex((index, iterator) => {
    val buffer = new ListBuffer[String]
    while (iterator.hasNext) {
        buffer.append(index + "分区:" + iterator.next())
    }
    buffer.toIterator
}).foreach(println)
```

执行结果如下：`help_keyword` 这张表只有 600 条左右的数据，本来数据应该均匀分布在 10 个分区，但是 0 分区里面却有 319 条数据，这是因为设置了下限，所有小于 300 的数据都会被限制在第一个分区，即 0 分区。同理所有大于 500 的数据被分配在 9 分区，即最后一个分区。

**写入数据** 

```java
val df = spark.read.format("json").load("/usr/file/json/emp.json")
df.write
.format("jdbc")
.option("url", "jdbc:mysql://127.0.0.1:3306/mysql")
.option("user", "root").option("password", "root")
.option("dbtable", "emp")
.save()
```

#### > 数据源 - CSV

CSV 是一种常见的文本文件格式，其中每一行表示一条记录，记录中的每个字段用逗号分隔。

**读取CSV文件**

自动推断类型读取读取示例：

```java
spark.read.format("csv")
.option("header", "false")        // 文件中的第一行是否为列的名称
.option("mode", "FAILFAST")      // 是否快速失败
.option("inferSchema", "true")   // 是否自动推断 schema
.load("/usr/file/csv/dept.csv")
.show()
```

使用预定义类型：

```java
import org.apache.spark.sql.types.{StructField, StructType, StringType,LongType}
//预定义数据格式
val myManualSchema = new StructType(Array(
    StructField("deptno", LongType, nullable = false),
    StructField("dname", StringType,nullable = true),
    StructField("loc", StringType,nullable = true)
))
spark.read.format("csv")
.option("mode", "FAILFAST")
.schema(myManualSchema)
.load("/usr/file/csv/dept.csv")
.show()
```

**写入CSV文件**

```java
df.write.format("csv").mode("overwrite").save("/tmp/csv/dept2")
```

也可以指定具体的分隔符：

```java
df.write.format("csv").mode("overwrite").option("sep", "\t").save("/tmp/csv/dept2")
```

#### > 数据源 - JSON

**读取JSON文件** 

```java
spark.read.format("json").option("mode", "FAILFAST").load("/usr/file/json/dept.json").show(5)
```

需要注意的是：默认不支持一条数据记录跨越多行 (如下)，可以通过配置 `multiLine` 为 `true` 来进行更改，其默认值为 `false`。

```json
// 默认支持单行
{"DEPTNO": 10,"DNAME": "ACCOUNTING","LOC": "NEW YORK"}

//默认不支持多行
{
  "DEPTNO": 10,
  "DNAME": "ACCOUNTING",
  "LOC": "NEW YORK"
}
```

**写入JSON文件** 

```java
df.write.format("json").mode("overwrite").save("/tmp/spark/json/dept")
```

#### > 数据源 - Parquet

 Parquet 是一个开源的面向列的数据存储，它提供了多种存储优化，允许读取单独的列非整个文件，这不仅节省了存储空间而且提升了读取效率，它是 Spark 是默认的文件格式。

**读取Parquet文件**

```java
spark.read.format("parquet").load("/usr/file/parquet/dept.parquet").show(5)
```

**写入Parquet文件**

```java
df.write.format("parquet").mode("overwrite").save("/tmp/spark/parquet/dept")
```

**可选配置**

Parquet 文件有着自己的存储规则，因此其可选配置项比较少，常用的有如下两个：

| 读写操作  | 配置项                  | 可选值                                      | 默认值                                    | 描述                                       |
| ----- | -------------------- | ---------------------------------------- | -------------------------------------- | ---------------------------------------- |
| Write | compression or codec | None,<br/>uncompressed,<br/>bzip2,<br/>deflate, gzip,<br/>lz4, or snappy | None                                   | 压缩文件格式                                   |
| Read  | mergeSchema          | true, false                              | 取决于配置项 `spark.sql.parquet.mergeSchema` | 当为真时，Parquet 数据源将所有数据文件收集的 Schema 合并在一起，否则将从摘要文件中选择 Schema，如果没有可用的摘要文件，则从随机数据文件中选择 Schema。 |

> 更多可选配置可以参阅官方文档：https://spark.apache.org/docs/latest/sql-data-sources-parquet.html

#### > 数据源 - ORC

ORC 是一种自描述的、类型感知的列文件格式，它针对大型数据的读写进行了优化，也是大数据中常用的文件格式。

**读取ORC文件**

```java
spark.read.format("orc").load("/usr/file/orc/dept.orc").show(5)
```

**写入ORC文件**

```java
spark.write.format("orc").mode("overwrite").save("/tmp/spark/orc/dept")
```

#### > 数据源 - Text

Text 文件在读写性能方面并没有任何优势，且不能表达明确的数据结构，所以其使用的比较少，读写操作如下：

**读取Text数据** 

```java
spark.read.textFile("/usr/file/txt/dept.txt").show()
```

**写入Text数据** 

```java
df.write.text("/tmp/spark/txt/dept")
```

#### >  数据读写高级特性

**并行读**

多个 Executors 不能同时读取同一个文件，但它们可以同时读取不同的文件。这意味着当您从一个包含多个文件的文件夹中读取数据时，这些文件中的每一个都将成为 DataFrame 中的一个分区，并由可用的 Executors 并行读取。

**并行写**

写入的文件或数据的数量取决于写入数据时 DataFrame 拥有的分区数量。默认情况下，每个数据分区写一个文件。

**分区写入**

分区和分桶这两个概念和 Hive 中分区表和分桶表是一致的。都是将数据按照一定规则进行拆分存储。需要注意的是 `partitionBy` 指定的分区和 RDD 中分区不是一个概念：这里的**分区表现为输出目录的子目录**，数据分别存储在对应的子目录中。

```java
val df = spark.read.format("json").load("/usr/file/json/emp.json")
df.write.mode("overwrite").partitionBy("deptno").save("/tmp/spark/partitions")
```

输出结果如下：可以看到输出被按照部门编号分为三个子目录，子目录中才是对应的输出文件。

![img](./images/spark-分区.png) 

**分桶写入**

分桶写入就是将数据按照指定的列和桶数进行散列，目前分桶写入只支持保存为表，实际上这就是 Hive 的分桶表。

```java
val numberBuckets = 10
val columnToBucketBy = "empno"
df.write.format("parquet").mode("overwrite")
.bucketBy(numberBuckets, columnToBucketBy).saveAsTable("bucketedFiles")
```

**文件大小管理**

如果写入产生小文件数量过多，这时会产生大量的元数据开销。Spark 和 HDFS 一样，都不能很好的处理这个问题，这被称为“small file problem”。同时数据文件也不能过大，否则在查询时会有不必要的性能开销，因此要把文件大小控制在一个合理的范围内。

在上文我们已经介绍过可以通过分区数量来控制生成文件的数量，从而间接控制文件大小。Spark 2.2 引入了一种新的方法，以更自动化的方式控制文件大小，这就是 `maxRecordsPerFile` 参数，它允许你通过控制写入文件的记录数来控制文件大小。

```java
 // Spark 将确保文件最多包含 5000 条记录
df.write.option(“maxRecordsPerFile”, 5000)
```

#### > 可选配置附录

**CSV读写可选配置** 

| 读\写操作 | 配置项                         | 可选值                                      | 默认值                           | 描述                                       |
| ----- | --------------------------- | ---------------------------------------- | ----------------------------- | ---------------------------------------- |
| Both  | seq                         | 任意字符                                     | `,`(逗号)                       | 分隔符                                      |
| Both  | header                      | true, false                              | false                         | 文件中的第一行是否为列的名称。                          |
| Read  | escape                      | 任意字符                                     | \                             | 转义字符                                     |
| Read  | inferSchema                 | true, false                              | false                         | 是否自动推断列类型                                |
| Read  | ignoreLeadingWhiteSpace     | true, false                              | false                         | 是否跳过值前面的空格                               |
| Both  | ignoreTrailingWhiteSpace    | true, false                              | false                         | 是否跳过值后面的空格                               |
| Both  | nullValue                   | 任意字符                                     | “”                            | 声明文件中哪个字符表示空值                            |
| Both  | nanValue                    | 任意字符                                     | NaN                           | 声明哪个值表示 NaN 或者缺省值                        |
| Both  | positiveInf                 | 任意字符                                     | Inf                           | 正无穷                                      |
| Both  | negativeInf                 | 任意字符                                     | -Inf                          | 负无穷                                      |
| Both  | compression or codec        | None,<br/>uncompressed,<br/>bzip2, deflate,<br/>gzip, lz4, or<br/>snappy | none                          | 文件压缩格式                                   |
| Both  | dateFormat                  | 任何能转换为 Java 的 <br/>SimpleDataFormat 的字符串 | yyyy-MM-dd                    | 日期格式                                     |
| Both  | timestampFormat             | 任何能转换为 Java 的 <br/>SimpleDataFormat 的字符串 | `yyyy-MM-dd'T'HH:mm:ss.SSSZZ` | 时间戳格式                                    |
| Read  | maxColumns                  | 任意整数                                     | 20480                         | 声明文件中的最大列数                               |
| Read  | maxCharsPerColumn           | 任意整数                                     | 1000000                       | 声明一个列中的最大字符数。                            |
| Read  | escapeQuotes                | true, false                              | true                          | 是否应该转义行中的引号。                             |
| Read  | maxMalformedLogPerPartition | 任意整数                                     | 10                            | 声明每个分区中最多允许多少条格式错误的数据，超过这个值后格式错误的数据将不会被读取 |
| Write | quoteAll                    | true, false                              | false                         | 指定是否应该将所有值都括在引号中，而不只是转义具有引号字符的值。         |
| Read  | multiLine                   | true, false                              | false                         | 是否允许每条完整记录跨域多行                           |

**JSON读写可选配置** 

| 读\写操作 | 配置项                                | 可选值                                      | 默认值                              |
| ----- | ---------------------------------- | ---------------------------------------- | -------------------------------- |
| Both  | compression or codec               | None,<br/>uncompressed,<br/>bzip2, deflate,<br/>gzip, lz4, or<br/>snappy | none                             |
| Both  | dateFormat                         | 任何能转换为 Java 的 SimpleDataFormat 的字符串      | yyyy-MM-dd                       |
| Both  | timestampFormat                    | 任何能转换为 Java 的 SimpleDataFormat 的字符串      | `yyyy-MM-dd'T'HH:mm:ss.SSSZZ`    |
| Read  | primitiveAsString                  | true, false                              | false                            |
| Read  | allowComments                      | true, false                              | false                            |
| Read  | allowUnquotedFieldNames            | true, false                              | false                            |
| Read  | allowSingleQuotes                  | true, false                              | true                             |
| Read  | allowNumericLeadingZeros           | true, false                              | false                            |
| Read  | allowBackslashEscapingAnyCharacter | true, false                              | false                            |
| Read  | columnNameOfCorruptRecord          | true, false                              | Value of spark.sql.column&NameOf |
| Read  | multiLine                          | true, false                              | false                            |

**数据库读写可选配置** 

| 属性名称                                     | 含义                                       |
| ---------------------------------------- | ---------------------------------------- |
| url                                      | 数据库地址                                    |
| dbtable                                  | 表名称                                      |
| driver                                   | 数据库驱动                                    |
| partitionColumn,<br/>lowerBound, upperBoun | 分区总数，上界，下界                               |
| numPartitions                            | 可用于表读写并行性的最大分区数。如果要写的分区数量超过这个限制，那么可以调用 coalesce(numpartition) 重置分区数。 |
| fetchsize                                | 每次往返要获取多少行数据。此选项仅适用于读取数据。                |
| batchsize                                | 每次往返插入多少行数据，这个选项只适用于写入数据。默认值是 1000。      |
| isolationLevel                           | 事务隔离级别：可以是 NONE，READ_COMMITTED, READ_UNCOMMITTED，REPEATABLE_READ 或 SERIALIZABLE，即标准事务隔离级别。<br/>默认值是 READ_UNCOMMITTED。这个选项只适用于数据读取。 |
| createTableOptions                       | 写入数据时自定义创建表的相关配置                         |
| createTableColumnTypes                   | 写入数据时自定义创建列的列类型                          |

> 数据库读写更多配置可以参阅官方文档：https://spark.apache.org/docs/latest/sql-data-sources-jdbc.html



### （4）Spark - SQL 的 Join 操作

- **数据准备**

本文主要介绍 Spark SQL 的多表连接，需要预先准备测试数据。分别创建员工和部门的 Datafame，并注册为临时视图，代码如下：

```java
val spark = SparkSession.builder().appName("aggregations").master("local[2]").getOrCreate()

val empDF = spark.read.json("/usr/file/json/emp.json")
empDF.createOrReplaceTempView("emp")

val deptDF = spark.read.json("/usr/file/json/dept.json")
deptDF.createOrReplaceTempView("dept")
```

两表的主要字段如下：

```properties
emp 员工表
 |-- ENAME: 员工姓名
 |-- DEPTNO: 部门编号
 |-- EMPNO: 员工编号
 |-- HIREDATE: 入职时间
 |-- JOB: 职务
 |-- MGR: 上级编号
 |-- SAL: 薪资
 |-- COMM: 奖金  
```

```properties
dept 部门表
 |-- DEPTNO: 部门编号
 |-- DNAME:  部门名称
 |-- LOC:    部门所在城市
```

- **连接类型**

Spark 中支持多种连接类型：

- **Inner Join** : 内连接；
- **Full Outer Join** :  全外连接；
- **Left Outer Join** :  左外连接；
- **Right Outer Join** :  右外连接；
- **Left Semi Join** :  左半连接；
- **Left Anti Join** :  左反连接；
- **Natural Join** :  自然连接；
- **Cross (or Cartesian) Join** :  交叉 (或笛卡尔) 连接。

其中内，外连接，笛卡尔积均与普通关系型数据库中的相同，如下图所示：

![img](./images/sql-join.jpg)

这里解释一下左半连接和左反连接，这两个连接等价于关系型数据库中的 `IN` 和 `NOT IN` 字句：

```sql
-- LEFT SEMI JOIN
SELECT * FROM emp LEFT SEMI JOIN dept ON emp.deptno = dept.deptno
-- 等价于如下的 IN 语句
SELECT * FROM emp WHERE deptno IN (SELECT deptno FROM dept)

-- LEFT ANTI JOIN
SELECT * FROM emp LEFT ANTI JOIN dept ON emp.deptno = dept.deptno
-- 等价于如下的 IN 语句
SELECT * FROM emp WHERE deptno NOT IN (SELECT deptno FROM dept)
```

所有连接类型的示例代码如下：

#### > INNER JOIN

```java
// 1.定义连接表达式
val joinExpression = empDF.col("deptno") === deptDF.col("deptno")
// 2.连接查询 
empDF.join(deptDF,joinExpression).select("ename","dname").show()

// 等价 SQL 如下：
spark.sql("SELECT ename,dname FROM emp JOIN dept ON emp.deptno = dept.deptno").show()
```

#### >  FULL OUTER JOIN

```java
empDF.join(deptDF, joinExpression, "outer").show()
spark.sql("SELECT * FROM emp FULL OUTER JOIN dept ON emp.deptno = dept.deptno").show()
```

#### >  LEFT OUTER JOIN

```java
empDF.join(deptDF, joinExpression, "left_outer").show()
spark.sql("SELECT * FROM emp LEFT OUTER JOIN dept ON emp.deptno = dept.deptno").show()
```

#### >  RIGHT OUTER JOIN

```java
empDF.join(deptDF, joinExpression, "right_outer").show()
spark.sql("SELECT * FROM emp RIGHT OUTER JOIN dept ON emp.deptno = dept.deptno").show()
```

#### >  LEFT SEMI JOIN

```java
empDF.join(deptDF, joinExpression, "left_semi").show()
spark.sql("SELECT * FROM emp LEFT SEMI JOIN dept ON emp.deptno = dept.deptno").show()
```

#### > LEFT ANTI JOIN

```java
empDF.join(deptDF, joinExpression, "left_anti").show()
spark.sql("SELECT * FROM emp LEFT ANTI JOIN dept ON emp.deptno = dept.deptno").show()
```

#### >  CROSS JOIN

```java
empDF.join(deptDF, joinExpression, "cross").show()
spark.sql("SELECT * FROM emp CROSS JOIN dept ON emp.deptno = dept.deptno").show()
```

#### >  NATURAL JOIN

自然连接是在两张表中寻找那些数据类型和列名都相同的字段，然后自动地将他们连接起来，并返回所有符合条件的结果。

```java
spark.sql("SELECT * FROM emp NATURAL JOIN dept").show()
```

以下是一个自然连接的查询结果，程序自动推断出使用两张表都存在的 dept 列进行连接，其实际等价于：

```java
spark.sql("SELECT * FROM emp JOIN dept ON emp.deptno = dept.deptno").show()
```

![img](./images/spark-sql-NATURAL-JOIN.png) 

由于自然连接常常会产生不可预期的结果，所以并不推荐使用。

#### >  连接的执行

在对大表与大表之间进行连接操作时，通常都会触发 `Shuffle Join`，两表的所有分区节点会进行 `All-to-All` 的通讯，这种查询通常比较昂贵，会对网络 IO 会造成比较大的负担。

![img](./images/spark-Big-table–to–big-table.png)

而对于大表和小表的连接操作，Spark 会在一定程度上进行优化，如果小表的数据量小于 Worker Node 的内存空间，Spark 会考虑将小表的数据广播到每一个 Worker Node，在每个工作节点内部执行连接计算，这可以降低网络的 IO，但会加大每个 Worker Node 的 CPU 负担。

![img](./images/spark-Big-table–to–small-table.png)

是否采用广播方式进行 `Join` 取决于程序内部对小表的判断，如果想明确使用广播方式进行 `Join`，则可以在 DataFrame API 中使用 `broadcast` 方法指定需要广播的小表：

```java
empDF.join(broadcast(deptDF), joinExpression).show()
```



### （5）Spark - SQL 的聚合参数

#### > 简单聚合

- **数据准备**

```java
// 需要导入 spark sql 内置的函数包
import org.apache.spark.sql.functions._

val spark = SparkSession.builder().appName("aggregations").master("local[2]").getOrCreate()
val empDF = spark.read.json("/usr/file/json/emp.json")
// 注册为临时视图，用于后面演示 SQL 查询
empDF.createOrReplaceTempView("emp")
empDF.show()
```

- **count**

```java
// 计算员工人数
empDF.select(count("ename")).show()
```

- **countDistinct** 

```java
// 计算姓名不重复的员工人数
empDF.select(countDistinct("deptno")).show()
```

- **approx_count_distinct**

通常在使用大型数据集时，你可能关注的只是近似值而不是准确值，这时可以使用 approx_count_distinct 函数，并可以使用第二个参数指定最大允许误差。

```java
empDF.select(approx_count_distinct ("ename",0.1)).show()
```

- **first & last**

获取 DataFrame 中指定列的第一个值或者最后一个值。

```java
empDF.select(first("ename"),last("job")).show()
```

- **min & max**

获取 DataFrame 中指定列的最小值或者最大值。

```java
empDF.select(min("sal"),max("sal")).show()
```

- **sum & sumDistinct**

求和以及求指定列所有不相同的值的和。

```java
empDF.select(sum("sal")).show()
empDF.select(sumDistinct("sal")).show()
```

- **avg**

内置的求平均数的函数。

```java
empDF.select(avg("sal")).show()
```

**数学函数**

Spark SQL 中还支持多种数学聚合函数，用于通常的数学计算，以下是一些常用的例子：

```java
// 1.计算总体方差、均方差、总体标准差、样本标准差
empDF.select(var_pop("sal"), var_samp("sal"), stddev_pop("sal"), stddev_samp("sal")).show()

// 2.计算偏度和峰度
empDF.select(skewness("sal"), kurtosis("sal")).show()

// 3. 计算两列的皮尔逊相关系数、样本协方差、总体协方差。(这里只是演示，员工编号和薪资两列实际上并没有什么关联关系)
empDF.select(corr("empno", "sal"), covar_samp("empno", "sal"),covar_pop("empno", "sal")).show()
```

- **聚合数据到集合**

```java
scala>  empDF.agg(collect_set("job"), collect_list("ename")).show()

输出：
+--------------------+--------------------+
|    collect_set(job)| collect_list(ename)|
+--------------------+--------------------+
|[MANAGER, SALESMA...|[SMITH, ALLEN, WA...|
+--------------------+--------------------+
```



#### > 分组聚合

- **简单分组**

```java
empDF.groupBy("deptno", "job").count().show()
//等价 SQL
spark.sql("SELECT deptno, job, count(*) FROM emp GROUP BY deptno, job").show()

输出：
+------+---------+-----+
|deptno|      job|count|
+------+---------+-----+
|    10|PRESIDENT|    1|
|    30|    CLERK|    1|
|    10|  MANAGER|    1|
|    30|  MANAGER|    1|
|    20|    CLERK|    2|
|    30| SALESMAN|    4|
|    20|  ANALYST|    2|
|    10|    CLERK|    1|
|    20|  MANAGER|    1|
+------+---------+-----+
```

- **分组聚合**

```java
empDF.groupBy("deptno").agg(count("ename").alias("人数"), sum("sal").alias("总工资")).show()
// 等价语法
empDF.groupBy("deptno").agg("ename"->"count","sal"->"sum").show()
// 等价 SQL
spark.sql("SELECT deptno, count(ename) ,sum(sal) FROM emp GROUP BY deptno").show()

输出：
+------+----+------+
|deptno|人数|总工资|
+------+----+------+
|    10|   3|8750.0|
|    30|   6|9400.0|
|    20|   5|9375.0|
+------+----+------+
```



#### > 自定义聚合函数

Scala 提供了两种自定义聚合函数的方法，分别如下：

- 有类型的自定义聚合函数，主要适用于 DataSet；
- 无类型的自定义聚合函数，主要适用于 DataFrame。

以下分别使用两种方式来自定义一个求平均值的聚合函数，这里以计算员工平均工资为例。两种自定义方式分别如下：

- **有类型的自定义函数**

```java
import org.apache.spark.sql.expressions.Aggregator
import org.apache.spark.sql.{Encoder, Encoders, SparkSession, functions}

// 1.定义员工类,对于可能存在 null 值的字段需要使用 Option 进行包装
case class Emp(ename: String, comm: scala.Option[Double], deptno: Long, empno: Long,
               hiredate: String, job: String, mgr: scala.Option[Long], sal: Double)

// 2.定义聚合操作的中间输出类型
case class SumAndCount(var sum: Double, var count: Long)

/* 3.自定义聚合函数
 * @IN  聚合操作的输入类型
 * @BUF reduction 操作输出值的类型
 * @OUT 聚合操作的输出类型
 */
object MyAverage extends Aggregator[Emp, SumAndCount, Double] {
    
    // 4.用于聚合操作的的初始零值
    override def zero: SumAndCount = SumAndCount(0, 0)
    
    // 5.同一分区中的 reduce 操作
    override def reduce(avg: SumAndCount, emp: Emp): SumAndCount = {
        avg.sum += emp.sal
        avg.count += 1
        avg
    }

    // 6.不同分区中的 merge 操作
    override def merge(avg1: SumAndCount, avg2: SumAndCount): SumAndCount = {
        avg1.sum += avg2.sum
        avg1.count += avg2.count
        avg1
    }

    // 7.定义最终的输出类型
    override def finish(reduction: SumAndCount): Double = reduction.sum / reduction.count

    // 8.中间类型的编码转换
    override def bufferEncoder: Encoder[SumAndCount] = Encoders.product

    // 9.输出类型的编码转换
    override def outputEncoder: Encoder[Double] = Encoders.scalaDouble
}

object SparkSqlApp {

    // 测试方法
    def main(args: Array[String]): Unit = {

        val spark = SparkSession.builder().appName("Spark-SQL").master("local[2]").getOrCreate()
        import spark.implicits._
        val ds = spark.read.json("file/emp.json").as[Emp]

        // 10.使用内置 avg() 函数和自定义函数分别进行计算，验证自定义函数是否正确
        val myAvg = ds.select(MyAverage.toColumn.name("average_sal")).first()
        val avg = ds.select(functions.avg(ds.col("sal"))).first().get(0)

        println("自定义 average 函数 : " + myAvg)
        println("内置的 average 函数 : " + avg)
    }
}
```

自定义聚合函数需要实现的方法比较多，这里以绘图的方式来演示其执行流程，以及每个方法的作用：

![img](./images/spark-sql-自定义函数.png)



关于 `zero`,`reduce`,`merge`,`finish` 方法的作用在上图都有说明，这里解释一下中间类型和输出类型的编码转换，这个写法比较固定，基本上就是两种情况：

- 自定义类型 Case Class 或者元组就使用 `Encoders.product` 方法；
- 基本类型就使用其对应名称的方法，如 `scalaByte `，`scalaFloat`，`scalaShort` 等，示例如下：

```java
override def bufferEncoder: Encoder[SumAndCount] = Encoders.product
override def outputEncoder: Encoder[Double] = Encoders.scalaDouble
```

- **无类型的自定义聚合函数**

理解了有类型的自定义聚合函数后，无类型的定义方式也基本相同，代码如下：

```java
import org.apache.spark.sql.expressions.{MutableAggregationBuffer, UserDefinedAggregateFunction}
import org.apache.spark.sql.types._
import org.apache.spark.sql.{Row, SparkSession}

object MyAverage extends UserDefinedAggregateFunction {
  // 1.聚合操作输入参数的类型,字段名称可以自定义
  def inputSchema: StructType = StructType(StructField("MyInputColumn", LongType) :: Nil)

  // 2.聚合操作中间值的类型,字段名称可以自定义
  def bufferSchema: StructType = {
    StructType(StructField("sum", LongType) :: StructField("MyCount", LongType) :: Nil)
  }

  // 3.聚合操作输出参数的类型
  def dataType: DataType = DoubleType

  // 4.此函数是否始终在相同输入上返回相同的输出,通常为 true
  def deterministic: Boolean = true

  // 5.定义零值
  def initialize(buffer: MutableAggregationBuffer): Unit = {
    buffer(0) = 0L
    buffer(1) = 0L
  }

  // 6.同一分区中的 reduce 操作
  def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
    if (!input.isNullAt(0)) {
      buffer(0) = buffer.getLong(0) + input.getLong(0)
      buffer(1) = buffer.getLong(1) + 1
    }
  }

  // 7.不同分区中的 merge 操作
  def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
    buffer1(0) = buffer1.getLong(0) + buffer2.getLong(0)
    buffer1(1) = buffer1.getLong(1) + buffer2.getLong(1)
  }

  // 8.计算最终的输出值
  def evaluate(buffer: Row): Double = buffer.getLong(0).toDouble / buffer.getLong(1)
}

object SparkSqlApp {

  // 测试方法
  def main(args: Array[String]): Unit = {

    val spark = SparkSession.builder().appName("Spark-SQL").master("local[2]").getOrCreate()
    // 9.注册自定义的聚合函数
    spark.udf.register("myAverage", MyAverage)

    val df = spark.read.json("file/emp.json")
    df.createOrReplaceTempView("emp")

    // 10.使用自定义函数和内置函数分别进行计算
    val myAvg = spark.sql("SELECT myAverage(sal) as avg_sal FROM emp").first()
    val avg = spark.sql("SELECT avg(sal) as avg_sal FROM emp").first()

    println("自定义 average 函数 : " + myAvg)
    println("内置的 average 函数 : " + avg)
  }
}
```



## 三、Spark-Streaming

### （1）Spark Streaming 概述

- **Spark Streaming 是什么**

  > Spark 流使得构建可扩展的容错流应用程序变得更加容易。
  >
  > Spark Streaming 用于流式数据的处理。Spark Streaming 支持的数据输入源很多，例如：Kafka、Flume、Twitter、ZeroMQ 和简单的 TCP 套接字等等。数据输入后可以用 Spark 的高度抽象原语
  > 如：map、reduce、join、window 等进行运算。而结果也能保存在很多地方，如 HDFS，数据库等。

![img](./images/streaming1.PNG)

> 在 Spark Streaming 中，处理数据的单位是一批而不是单条，而数据采集却是逐条进行的，因此 Spark Streaming 系统需要设置间隔使得数据汇总到一定的量后再一并操作，这个间隔就是批处理间隔。批处理间隔是 Spark Streaming 的核心概念和关键参数，它决定了 Spark Streaming 提交作业的频率和数据处理的延迟，同时也影响着数据处理的吞吐量和性能。

![img](./images/streaming2.PNG)

> Spark Streaming 提供了一个高级抽象: discretized stream(SStream), DStream 表示一个连续的数据流.
>
> DStream 可以由来自数据源的输入数据流来创建, 也可以通过在其他的 DStream 上应用一些高阶操作来得到.在内部, 一个 DSteam 是由一个个 RDD 序列来表示的。

- **特点**
  - 易用：通过高阶函数来构建应用
  - 容错
  - 易整合到 Spark  体系中
  - 缺点：Spark Streaming 是一种“微量批处理”架构, 和其他基于“一次处理一条记录”架构的系统相比, 它的延迟会相对高一些
- **Spark-Streaming架构** 

![img](./images/架构.PNG)

- **背压机制**

> 为了更好的协调数据接收速率与资源处理能力，1.5 版本开始 Spark Streaming 可以动态控制数据接收速率来适配集群数据处理能力。背压机制（即 Spark StreamingBackpressure）: 根据 JobScheduler 反馈作业的执行信息来动态调整 Receiver 数据接收率。
>
> 通过属性 spark.streaming.backpressure.enabled 来控制是否启用 backpressure 机制，默认值 false ，即不启用。



### （2）DStream 入门 

- **WordCount词频统计** 

  **需求：**

  &emsp;使用 netcat 工具向 9999 端口不断的发送数据，通过 SparkStreaming 读取端口数据并统计不同单词出现的次数

  - **基本数据源**：包括文件系统、Socket 连接等；
  - **高级数据源**：包括 Kafka，Flume，Kinesis 等。

  **添加依赖**:

  ```properties
  <dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming_2.12</artifactId>
    <version>3.0.0</version>
  </dependency>
  ```

  **编写代码**：

  ```java
  object StreamWordCount {
    def main(args: Array[String]): Unit = {
      
      //1.初始化 Spark 配置信息
      val sparkConf = new
      SparkConf().setMaster("local[*]").setAppName("StreamWordCount")
        
      //2.初始化 SparkStreamingContext
      val ssc = new StreamingContext(sparkConf, Seconds(3))
        
      //3.通过监控端口创建 DStream，读进来的数据为一行行
      val lineStreams = ssc.socketTextStream("linux1", 9999)
        
      //将每一行数据做切分，形成一个个单词
      val wordStreams = lineStreams.flatMap(_.split(" "))
        
      //将单词映射成元组（word,1）
      val wordAndOneStreams = wordStreams.map((_, 1))
        
      //将相同的单词次数做统计
      val wordAndCountStreams = wordAndOneStreams.reduceByKey(_+_)
        
      //打印
      wordAndCountStreams.print()
        
      //启动 SparkStreamingContext
      ssc.start()
      ssc.awaitTermination()
    }
  }
  ```

  > 在示例代码中，使用 `streamingContext.start()` 代表启动服务，此时还要使用 `streamingContext.awaitTermination()` 使服务处于等待和可用的状态，直到发生异常或者手动使用 `streamingContext.stop()` 进行终止。

  **启动程序并通过 netcat 发送数据**

  ```shell
  nc -lk 9999
  hello spark
  ```

  > 在内部实现上，DStream 是一系列连续的 RDD 来表示。每个 RDD 含有一段时间间隔内的数据。



### （3）DStream 创建   

&emsp;测试过程中，可以通过使用 ssc.queueStream(queueOfRDDs)来创建 DStream，每一个推送到这个队列中的 RDD，都会作为一个 DStream 处理。

➢  需求：循环创建几个 RDD，将 RDD 放入队列。通过 SparkStream 创建 Dstream，计算WordCount。

**代码**： 

```java
object RDDStream {
  def main(args: Array[String]) {
    
    //1.初始化 Spark 配置信息
    val conf = new SparkConf().setMaster("local[*]").setAppName("RDDStream")
      
    //2.初始化 SparkStreamingContext
    val ssc = new StreamingContext(conf, Seconds(4))
      
    //3.创建 RDD 队列
    val rddQueue = new mutable.Queue[RDD[Int]]()
      
    //4.创建 QueueInputDStream
    val inputStream = ssc.queueStream(rddQueue,oneAtATime = false)
      
    //5.处理队列中的 RDD 数据
    val mappedStream = inputStream.map((_,1))
    val reducedStream = mappedStream.reduceByKey(_ + _)
      
    //6.打印结果
    reducedStream.print()
      
    //7.启动任务
    ssc.start()
      
    //8.循环创建并向 RDD 队列中放入 RDD
    for (i <- 1 to 5) {
      rddQueue += ssc.sparkContext.makeRDD(1 to 300, 10)
      Thread.sleep(2000)
    }
    
    ssc.awaitTermination()
  }
}
```

**结果展示**：

```java
-------------------------------------------
Time: 1539075280000 ms
-------------------------------------------
(4,60)
(0,60)
(6,60)
(8,60)
(2,60)
(1,60)
(3,60)
(7,60)
(9,60)
(5,60)
-------------------------------------------
Time: 1539075284000 ms
-------------------------------------------
(4,60)
(0,60)
(6,60)
(8,60)
(2,60)
(1,60)
(3,60)
(7,60)
(9,60)
(5,60)
-------------------------------------------
Time: 1539075288000 ms
-------------------------------------------
(4,30)
(0,30)
(6,30)
(8,30)
(2,30)
(1,30)
(3,30)
(7,30)
(9,30)
(5,30)
-------------------------------------------
Time: 1539075292000 ms
-------------------------------------------
```

### （4）DStream 转换 

> `DStream` 上的操作与 `RDD` 的类似，分为 `Transformations`（转换）和 `Output Operations`（输出）两种，此外转换操作中还有一些比较特殊的原语，如：`updateStateByKey()`、`transform()`以及各种 `Window `相关的原语

- **无状态转化操作** 

> 无状态转化操作就是把简单的 `RDD` 转化操作应用到每个批次上，也就是转化` DStream` 中的每一个 `RDD`。部分无状态转化操作列在了下表中。注意，针对键值对的` DStream` 转化操作(比如`reduceByKey())`要添加 `import StreamingContext._`才能在 `Scala` 中使用。

![img](./images/无状态转换.PNG)

> 需要记住的是，尽管这些函数看起来像作用在整个流上一样，但事实上每个 DStream 在内部是由许多 RDD（批次）组成，且无状态转化操作是分别应用到每个 RDD 上的。

- **有状态转化操作** 

  - **UpdateStateByKey** 

    > UpdateStateByKey 原语用于记录历史记录，有时，我们需要在 DStream 中跨批次维护状态(例如流计算中累加 wordcount)。针对这种情况，updateStateByKey()为我们提供了对一个状态变量的访问，用于键值对形式的 DStream。给定一个由(键，事件)对构成的 DStream，并传递一个指定如何根据新的事件更新每个键对应状态的函数，它可以构建出一个新的 DStream，其内部数据为(键，状态) 对。
    >
    > updateStateByKey() 的结果会是一个新的 DStream，其内部的 RDD 序列是由每个时间区间对应的(键，状态)对组成的。
    >
    > updateStateByKey 操作使得我们可以在用新信息进行更新时保持任意的状态。为使用这个功能，需要做下面两步：
    >
    > 1. 定义状态，状态可以是一个任意的数据类型。
    > 2. 定义状态更新函数，用此函数阐明如何使用之前的状态和来自输入流的新值对状态进行更新。
    > 3. 使用 updateStateByKey 需要对检查点目录进行配置，会使用检查点来保存状态。

    **编写代码**：

    ```java
    object WorldCount {
      def main(args: Array[String]) {
        
        // 定义更新状态方法，参数 values 为当前批次单词频度，state 为以往批次单词频度
        val updateFunc = (values: Seq[Int], state: Option[Int]) => {
          val currentCount = values.foldLeft(0)(_ + _)
          val previousCount = state.getOrElse(0)
          Some(currentCount + previousCount)
        }
        
        val conf = new
        SparkConf().setMaster("local[*]").setAppName("NetworkWordCount")
          
        val ssc = new StreamingContext(conf, Seconds(3))
          
        ssc.checkpoint("./ck")
          
        // Create a DStream that will connect to hostname:port, like hadoop102:9999
        val lines = ssc.socketTextStream("linux1", 9999)
          
        // Split each line into words
        val words = lines.flatMap(_.split(" "))
          
        // Count each word in each batch
        val pairs = words.map(word => (word, 1))
          
        // 使用 updateStateByKey 来更新状态，统计从运行开始以来单词总的次数
        val stateDstream = pairs.updateStateByKey[Int](updateFunc)
          
        stateDstream.print()
          
        ssc.start() // Start the computation
        ssc.awaitTermination() // Wait for the computation to terminate
        //ssc.stop()
      }
    }
    ```

    **启动程序并向 9999 端口发送数据**:

    ```shell
    nc -lk 9999
    Hello World
    Hello Scala
    ```

    **结果展示**:

    ```shell
    -------------------------------------------
    Time: 1504685175000 ms
    -------------------------------------------
    -------------------------------------------
    Time: 1504685181000 ms
    -------------------------------------------
    (shi,1)
    (shui,1)
    (ni,1)
    -------------------------------------------
    Time: 1504685187000 ms
    -------------------------------------------
    (shi,1)
    (ma,1)
    (hao,1)
    (shui,1)
    ```

    ​

  - **WindowOperations** 

    > Window Operations 可以设置窗口的大小和滑动窗口的间隔来动态的获取当前 Steaming 的允许状态。所有基于窗口的操作都需要两个参数，分别为窗口时长以及滑动步长。
    >
    > ➢  窗口时长：计算内容的时间范围；
    >
    > ➢  滑动步长：隔多久触发一次计算。
    >
    > 注意：这两者都必须为采集周期大小的整数倍。

### （5）DStream 输出

> 输出操作指定了对流数据经转化操作得到的数据所要执行的操作(例如把结果推入外部数据库或输出到屏幕上)。与 RDD 中的惰性求值类似，如果一个 DStream 及其派生出的 DStream 都没有被执行输出操作，那么这些 DStream 就都不会被求值。如果 StreamingContext 中没有设定输出操作，整个 context 就都不会启动。
>
> ➢  print()：在运行流程序的驱动结点上打印 DStream 中每一批次数据的最开始 10 个元素。这用于开发和调试。在 Python API 中，同样的操作叫 print()。
>
> ➢  saveAsTextFiles(prefix, [suffix])：以 text 文件形式存储这个 DStream 的内容。每一批次的存储文件名基于参数中的 prefix 和 suffix。”prefix-Time_IN_MS[.suffix]”。
>
> ➢  saveAsObjectFiles(prefix, [suffix])：以 Java 对象序列化的方式将 Stream 中的数据保存为SequenceFiles . 每一批次的存储文件名基于参数中的为"prefix-TIME_IN_MS[.suffix]". Python中目前不可用。
>
> ➢  saveAsHadoopFiles(prefix, [suffix])：将 Stream 中的数据保存为 Hadoop files. 每一批次的存储文件名基于参数中的为"prefix-TIME_IN_MS[.suffix]"。Python API 中目前不可用。
>
> ➢  foreachRDD(func)：这是最通用的输出操作，即将函数 func 用于产生于 stream 的每一个RDD。其中参数传入的函数 func 应该实现将每一个 RDD 中数据推送到外部系统，如将RDD 存入文件或者通过网络将其写入数据库。
>
> <br/>
>
> 通用的输出操作 foreachRDD()，它用来对 DStream 中的 RDD 运行任意计算。这和 transform()有些类似，都可以让我们访问任意 RDD。在 foreachRDD()中，可以重用我们在 Spark 中实现的所有行动操作。比如，常见的用例之一是把数据写到诸如 MySQL 的外部数据库中。
>
> 注意：
>
> 1)  连接不能写在 driver 层面（序列化）
>
> 2)  如果写在 foreach 则每个 RDD 中的每一条数据都创建，得不偿失；
>
> 3)  增加 foreachPartition，在分区创建（获取）。

### （6）Spark-straming 整和 kafka

- **导入依赖**

  ```properties
  <dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming_2.11</artifactId>
    <version>2.3.2</version>
    <scope>provided</scope>
  </dependency>
    
  <dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming-kafka-0-10_2.11</artifactId>
    <version>2.3.2}</version>
  </dependency>
  ```

  - **代码** 

  ```java
  public class SparkStreamingProcess {

      public static void main(String[] args) {
  		//=============kafka配置================
  		Map<String, Object> kafkaParams = new HashMap<>();
  		//Kafka服务监听端口
  		kafkaParams.put("bootstrap.servers", "slave01:9092");
  		//指定kafka输出key的数据类型及编码格式（默认为字符串类型编码格式为uft-8）
  		kafkaParams.put("key.deserializer", StringDeserializer.class);
  		//指定kafka输出value的数据类型及编码格式（默认为字符串类型编码格式为uft-8）
  		kafkaParams.put("value.deserializer", StringDeserializer.class);
  		//消费者ID，随意指定
  		kafkaParams.put("group.id", "test_id");
  		//指定从latest(最新,其他版本的是largest这里不行)还是smallest(最早)处开始读取数据
  		kafkaParams.put("auto.offset.reset", "latest");
  		//如果true,consumer定期地往zookeeper写入每个分区的offset
  		kafkaParams.put("enable.auto.commit", false);

  		//kafka - topic
  		Collection<String> topics = Collections.singletonList("test");

  		//构建SparkStreaming上下文
  		SparkConf conf = new SparkConf().setMaster("yarn").setAppName("SparkStreaming-kafka");  //线上使用 - yarn ; 本地使用 - local[*]
  		
           //每隔5秒钟，sparkStreaming作业就会收集最近5秒内的数据源接收过来的数据
  		JavaStreamingContext jssc = new JavaStreamingContext(conf, Durations.seconds(5));

  		//获取kafka的数据
  		JavaInputDStream<ConsumerRecord<String, String>> stream =
  				KafkaUtils.createDirectStream(
  						jssc,
  						LocationStrategies.PreferConsistent(),
  						ConsumerStrategies.Subscribe(topics, kafkaParams)
  				);

  		//foreachRDD：作用于DStream中每一个时间间隔的RDD;
  		//foreachPartition：作用于每一个时间间隔的RDD中的每一个partition;
  		//foreach：作用于每一个时间间隔的RDD中的每一个元素。
          stream.foreachRDD((VoidFunction<JavaRDD<ConsumerRecord<String, String>>>) rdd ->
                  rdd.foreachPartition((VoidFunction<Iterator<ConsumerRecord<String, String>>>) partition_iter -> {
                  while (partition_iter.hasNext()) {
                  	........
                  	........
                  }
         }));

          jssc.start();
          try {
              jssc.awaitTermination();
          } catch (Exception e) {
              // TODO Auto-generated catch block
              e.printStackTrace();
          }
      }
  ```

  ​

## 四、Spark 内核

loading.............



## 五、Spark 性能优化

loading.............



## 六、基于ZooKeeper搭建Spark高可用集群

### （1）集群规划

这里搭建一个 3 节点的 Spark 集群，其中三台主机上均部署 `Worker` 服务。同时为了保证高可用，除了在 hadoop001 上部署主 `Master` 服务外，还在 hadoop002 和 hadoop003 上分别部署备用的 `Master` 服务，Master 服务由 Zookeeper 集群进行协调管理，如果主 `Master` 不可用，则备用 `Master` 会成为新的主 `Master`。

![img](./images/spark集群规划.png)

### （2）前置条件

搭建 Spark 集群前，需要保证 JDK 环境、Zookeeper 集群和 Hadoop 集群已经搭建。

### （3）Spark集群搭建

#### 下载解压

下载所需版本的 Spark，官网下载地址：http://spark.apache.org/downloads.html

![img](./images/spark-download.png)

下载后进行解压：

```shell
tar -zxvf  spark-2.2.3-bin-hadoop2.6.tgz
```

#### 配置环境变量

```shell
vim /etc/profile
```

添加环境变量：

```shell
export SPARK_HOME=/usr/app/spark-2.2.3-bin-hadoop2.6
export  PATH=${SPARK_HOME}/bin:$PATH
```

使得配置的环境变量立即生效：

```shell
source /etc/profile
```

#### 集群配置

进入 `${SPARK_HOME}/conf` 目录，拷贝配置样本进行修改：

**1. spark-env.sh**

```shell
 cp spark-env.sh.template spark-env.sh
```

```shell
# 配置JDK安装位置
JAVA_HOME=/usr/java/jdk1.8.0_201
# 配置hadoop配置文件的位置
HADOOP_CONF_DIR=/usr/app/hadoop-2.6.0-cdh5.15.2/etc/hadoop
# 配置zookeeper地址
SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=hadoop001:2181,hadoop002:2181,hadoop003:2181 -Dspark.deploy.zookeeper.dir=/spark"
```

**2. slaves**

```shell
cp slaves.template slaves
```

配置所有 Woker 节点的位置：

```properties
hadoop001
hadoop002
hadoop003
```

#### 安装包分发

将 Spark 的安装包分发到其他服务器，分发后建议在这两台服务器上也配置一下 Spark 的环境变量。

```shell
scp -r /usr/app/spark-2.4.0-bin-hadoop2.6/   hadoop002:usr/app/
scp -r /usr/app/spark-2.4.0-bin-hadoop2.6/   hadoop003:usr/app/
```



### （4）启动集群

#### 启动ZooKeeper集群

分别到三台服务器上启动 ZooKeeper 服务：

```shell
 zkServer.sh start
```

#### 启动Hadoop集群

```shell
# 启动dfs服务
start-dfs.sh
# 启动yarn服务
start-yarn.sh
```

#### 启动Spark集群

进入 hadoop001 的 ` ${SPARK_HOME}/sbin` 目录下，执行下面命令启动集群。执行命令后，会在 hadoop001 上启动 `Maser` 服务，会在 `slaves` 配置文件中配置的所有节点上启动 `Worker` 服务。

```shell
start-all.sh
```

分别在 hadoop002 和 hadoop003 上执行下面的命令，启动备用的 `Master` 服务：

```shell
# ${SPARK_HOME}/sbin 下执行
start-master.sh
```

#### 查看服务

查看 Spark 的 Web-UI 页面，端口为 `8080`。此时可以看到 hadoop001 上的 Master 节点处于 `ALIVE` 状态，并有 3 个可用的 `Worker` 节点。

![img](./images/spark-集群搭建1.png)

而 hadoop002 和 hadoop003 上的 Master 节点均处于 `STANDBY` 状态，没有可用的 `Worker` 节点。

![img](./images/spark-集群搭建2.png)

![img](./images/spark-集群搭建3.png)



### （5）验证集群高可用

此时可以使用 `kill` 命令杀死 hadoop001 上的 `Master` 进程，此时备用 `Master` 会中会有一个再次成为 ` 主 Master`，我这里是 hadoop002，可以看到 hadoop2 上的 `Master` 经过 `RECOVERING` 后成为了新的主 `Master`，并且获得了全部可以用的 `Workers`。

![img](./images/spark-集群搭建4.png)

Hadoop002 上的 `Master` 成为主 `Master`，并获得了全部可以用的 `Workers`。

![img](./images/spark-集群搭建5.png)

此时如果你再在 hadoop001 上使用 `start-master.sh` 启动 Master 服务，那么其会作为备用 `Master` 存在。

### （6）提交作业

和单机环境下的提交到 Yarn 上的命令完全一致，这里以 Spark 内置的计算 Pi 的样例程序为例，提交命令如下：

```shell
spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
--deploy-mode client \
--executor-memory 1G \
--num-executors 10 \
/usr/app/spark-2.4.0-bin-hadoop2.6/examples/jars/spark-examples_2.11-2.4.0.jar \
100
```












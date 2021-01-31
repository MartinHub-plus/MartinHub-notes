![img](./images/sqoop.png)

##  一、Sqoop  安装

### （1）前提条件

- JDK安装
- Hadoop安装

> 具体的安装步骤可参考前面的章节



### （2）下载并解压

1) 下载地址：http://mirrors.hust.edu.cn/apache/sqoop/1.4.6/

2) 上传安装包 `sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz` 到服务器中

3) 解压 sqoop 安装包到指定目录，如：

```shell
$ tar -zxf sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz -C /opt/module/
```



### （3）修改配置文件

Sqoop 的配置文件与大多数大数据框架类似，在 sqoop 根目录下的 conf 目录中。

1) **重命名配置文件**

```shell
$ mv sqoop-env-template.sh sqoop-env.sh
```

2)  **修改配置文件**

sqoop-env.sh

```shell
export HADOOP_COMMON_HOME=/opt/module/hadoop-2.7.2
export HADOOP_MAPRED_HOME=/opt/module/hadoop-2.7.2
export HIVE_HOME=/opt/module/hive
export ZOOKEEPER_HOME=/opt/module/zookeeper-3.4.10
export ZOOCFGDIR=/opt/module/zookeeper-3.4.10/conf
export HBASE_HOME=/opt/module/hbase-1.3.1
```



### （4）拷贝 JDBC  驱动

拷贝 jdbc 驱动到 sqoop 的 lib 目录下，如：

```shell
$ cp mysql-connector-java-5.1.27-bin.jar /opt/module/sqoop-1.4.6.bin__hadoop-2.0.4-alpha/lib/
```



### （5）验证 Sqoop

我们可以通过某一个 command 来验证 sqoop 配置是否正确：

```shell
$ bin/sqoop help
```

出现一些 Warning 警告（警告信息已省略），并伴随着帮助命令的输出：

```shell
Available commands:
codegen Generate code to interact with database records
create-hive-table Import a table definition into Hive
eval Evaluate a SQL statement and display the results
export Export an HDFS directory to a database table
help List available commands
import Import a table from a database to HDFS
import-all-tables Import tables from a database to HDFS
import-mainframe Import datasets from a mainframe server to HDFS
job Work with saved jobs
list-databases List available databases on a server
list-tables List available tables in a database
merge Merge results of incremental imports
metastore Run a standalone Sqoop metastore
version Display version information
```



### （6）测试 Sqoop  是否能够成功连接数据库

```shell
$ bin/sqoop list-databases --connect jdbc:mysql://hadoop102:3306/ --username root --password 123456
```

出现如下输出：	

```shell
information_schema
metastore
mysql
oozie
performance_schema
```










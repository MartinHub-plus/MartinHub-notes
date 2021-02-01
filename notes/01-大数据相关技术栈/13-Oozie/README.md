## 一、Oozie 的安装部署

### （1）部署 Hadoop （CDH  版本的） 

#### > 修改 Hadoop  配置

- core-site.xml

  ```properties
  <!-- Oozie Server 的 Hostname -->
  <property>
  <name>hadoop.proxyuser.root.hosts</name>
  <value>*</value>
  </property>
  <!-- 允许被 Oozie 代理的用户组 -->
  <property>
  <name>hadoop.proxyuser.root.groups</name>
  <value>*</value>
  </property>
  ```

- mapred-site.xml

  ```properties
  <!-- 配置 MapReduce JobHistory Server 地址 ，默认端口 10020 -->
  <property>
  <name>mapreduce.jobhistory.address</name>
  <value>hadoop102:10020</value>
  </property>

  <!-- 配置 MapReduce JobHistory Server web ui 地址， 默认端口 19888 -->
  <property>
  <name>mapreduce.jobhistory.webapp.address</name>
  <value>hadoop102:19888</value>
  </property>
  ```

- yarn-site.xml

  ```properties
  <!-- 任务历史服务 -->
  <property>
  <name>yarn.log.server.url</name>
  <value>http://hadoop102:19888/jobhistory/logs/</value>
  </property>
  ```

  <font color='red'>完成后：记得 scp 同步到其他机器节点</font>

  ​

#### > 重启 Hadoop 集群

```shell
[root@hadoop102 hadoop-2.7.2]$ sbin/start-dfs.sh
[root@hadoop103 hadoop-2.7.2]$ sbin/start-yarn.sh
[root@hadoop102 hadoop-2.7.2]$ sbin/mr-jobhistory-daemon.sh start historyserver
```

<font color='red'>注意</font>：需要开启 JobHistoryServer, 最好执行一个 MR 任务进行测试。



### （2）部署 Oozie

#### > 解压 Oozie

```shell
[root@hadoop102 software]$ tar -zxvf /opt/software/cdh/oozie-4.0.0-cdh5.3.6.tar.gz -C ./
```

#### > 在 oozie  根目录下压 解压 oozie-hadooplibs-4.0.0-cdh5.3.6.tar.gz

```shell
[root@hadoop102 oozie-4.0.0-cdh5.3.6]$ tar -zxvf oozie-hadooplibs-4.0.0-cdh5.3.6.tar.gz
-C ../
```

完成后 Oozie 目录下会出现 hadooplibs 目录。

#### > 在 Oozie  目录下创建 libext目录 

```shell
[root@hadoop102 oozie-4.0.0-cdh5.3.6]$ mkdir libext/
```

#### > 拷贝依赖的 Jar 包 

1）将 hadooplibs 里面的 jar 包，拷贝到 libext 目录下：

```shell
[root@hadoop102 oozie-4.0.0-cdh5.3.6]$  cp  -ra hadooplibs/hadooplib-2.5.0-cdh5.3.6.oozie-4.0.0-cdh5.3.6/* libext/
```

2）拷贝 Mysql 驱动包到 libext 目录下：

```shell
[root@hadoop102 oozie-4.0.0-cdh5.3.6]$  cp  -a /opt/software/mysql-connector-java-5.1.27/mysql-connector-java-5.1.27-bin.jar ./libext/
```

#### > 将 ext-2.2.zip  拷贝到 libext/

ext 是一个 js 框架，用于展示 oozie 前端页面：

```shell
[root@hadoop102 oozie-4.0.0-cdh5.3.6]$ cp -a /opt/software/cdh/ext-2.2.zip libext/
```

#### > 修改 Oozie 配置文件

- oozie-site.xml

  ```properties
  属性：oozie.service.JPAService.jdbc.driver
  属性值：com.mysql.jdbc.Driver
  解释：JDBC 的驱动
  属性：oozie.service.JPAService.jdbc.url
  属性值：jdbc:mysql://hadoop102:3306/oozie
  解释：oozie 所需的数据库地址
  属性：oozie.service.JPAService.jdbc.username
  属性值：root
  解释：数据库用户名
  属性：oozie.service.JPAService.jdbc.password
  属性值：000000
  解释：数据库密码
  属性：oozie.service.HadoopAccessorService.hadoop.configurations
  属性值：*=/opt/module/cdh/hadoop-2.5.0-cdh5.3.6/etc/hadoop
  解释：让 Oozie 引用 Hadoop 的配置文件
  ```

#### > 在 Mysql  中创建 Oozie 的数据库 

进入 Mysql 并创建 oozie 数据库：

```shell
$ mysql -uroot -p000000
mysql> create database oozie;
```

#### >   初始化 Oozie

1)  上传 Oozie  目录下的 yarn.tar.gz  文件到 HDFS ：

<font color='red'>提示</font>：yarn.tar.gz 文件会自行解压

```shell
[root@hadoop102  oozie-4.0.0-cdh5.3.6]$  bin/oozie-setup.sh  sharelib  create  -fs
hdfs://hadoop102:8020 -locallib oozie-sharelib-4.0.0-cdh5.3.6-yarn.tar.gz
```

执行成功之后，去 50070 检查对应目录有没有文件生成。

2)  创建 oozie.sql  文件

```shell
[root@hadoop102 oozie-4.0.0-cdh5.3.6]$ bin/ooziedb.sh create -sqlfile oozie.sql -run
```

3)  打包项目，生成 war  包

```shell
[root@hadoop102 oozie-4.0.0-cdh5.3.6]$ bin/oozie-setup.sh prepare-war
```

#### > Oozie  的启动与关闭

启动命令如下：

```shell
[root@hadoop102 oozie-4.0.0-cdh5.3.6]$ bin/oozied.sh start
```

关闭命令如下：

```shell
[root@hadoop102 oozie-4.0.0-cdh5.3.6]$ bin/oozied.sh stop
```

#### > 访问 Oozie 的 的 Web  页面

http://hadoop102:11000/oozie





<div align="center"> <img  src="https://gitee.com/MartinHub/MartinHub-notes/raw/master/images/weixin.png" width="200"/> </div>












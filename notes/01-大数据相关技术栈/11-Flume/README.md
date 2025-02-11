![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/11-Flume/images/flume.png)

## 一、Flume 的安装部署

### （1）前置条件

Flume 需要依赖 JDK 1.8+，JDK 安装方式：

- 下载并解压

  &emsp;&emsp;在[官网](https://www.oracle.com/technetwork/java/javase/downloads/index.html) 下载所需版本的 JDK，这里我下载的版本为[JDK 1.8](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) ,下载后进行解压：

  ```shell
  [root@ java]# tar -zxvf jdk-8u201-linux-x64.tar.gz
  ```

  设置环境变量

  ```shell
  [root@ java]# vi /etc/profile
  ```

  &emsp;&emsp;添加如下配置：

  ```shell
  export JAVA_HOME=/usr/java/jdk1.8.0_201  
  export JRE_HOME=${JAVA_HOME}/jre  
  export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
  export PATH=${JAVA_HOME}/bin:$PATH
  ```

  &emsp;&emsp;执行 `source` 命令，使得配置立即生效：

  ```shell
  [root@ java]# source /etc/profile
  ```

  检查是否成功

  ```shell
  [root@ java]# java -version
  ```

  &emsp;&emsp;显示出对应的版本信息则代表安装成功。

  ```shell
  java version "1.8.0_201"
  Java(TM) SE Runtime Environment (build 1.8.0_201-b09)
  Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, mixed mode)
  ```

### （2）安装步骤

#### 2.1 下载并解压

下载所需版本的 Flume，这里我下载的是 `CDH` 版本的 Flume。下载地址为：http://archive.cloudera.com/cdh5/cdh/5/

```shell
# 下载后进行解压
tar -zxvf  flume-ng-1.6.0-cdh5.15.2.tar.gz
```

#### 2.2 配置环境变量

```shell
# vim /etc/profile
```

添加环境变量：

```shell
export FLUME_HOME=/usr/app/apache-flume-1.6.0-cdh5.15.2-bin
export PATH=$FLUME_HOME/bin:$PATH
```

使得配置的环境变量立即生效：

```shell
# source /etc/profile
```

#### 2.3 修改配置

进入安装目录下的 `conf/` 目录，拷贝 Flume 的环境配置模板 `flume-env.sh.template`：

```shell
# cp flume-env.sh.template flume-env.sh
```

修改 `flume-env.sh`,指定 JDK 的安装路径：

```shell
# Enviroment variables can be set here.
export JAVA_HOME=/usr/java/jdk1.8.0_201
```

#### 2.4 验证

由于已经将 Flume 的 bin 目录配置到环境变量，直接使用以下命令验证是否配置成功：

```shell
# flume-ng version
```

出现对应的版本信息则代表配置成功。

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/11-Flume/images/flume-version.png)

<div align="center"> <img  src="https://gitee.com/MartinHub/MartinHub-notes/raw/master/images/weixin.png" width="200"/> </div>
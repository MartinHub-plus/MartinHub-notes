## 一、RabbitMQ 单机环境搭建

### （1）前置条件

RabbitMQ 由 Erlang 语言所编写，所以在安装 RabbitMQ 前需要安装 Erlang 。两者的版本兼容关系如下。本篇文章选用的 RabbitMQ 版本为 3.7.15 ， Erlang 版本为 22.0 ：

| RabbitMQ version | Minimum required Erlang/OTP | Maximum supported Erlang/OTP |
| :--------------- | :-------------------------- | :--------------------------- |
| 3.7.15           | 20.3.x                      | 22.0.x                       |
| 3.7.7 ~ 3.7.14   | 20.3.x                      | 21.3.x                       |
| 3.7.0 ~ 3.7.6    | 19.3                        | 20.3.x                       |

> 表格来源：https://www.rabbitmq.com/which-erlang.html



### （2）Erlang 安装

#### > 下载并解压

Erlang 源码包下载地址：http://erlang.org/download/ ，下载后进行解压：

```shell
# 下载
wget http://erlang.org/download/otp_src_22.0.tar.gz
# 解压
tar -zxvf otp_src_22.0.tar.gz
```

#### > 编译和安装

Erlang 的编译过程中使用到了 `ncurses-devel` 库，需要预先安装：

```
yum install ncurses-devel
```

进入解压后的根目录：

```shell
# 配置安装目录
./configure --prefix=/usr/app/erlang
# 编译
make
# 安装
make install
```

#### > 验证安装结果

进入安装目录的 bin 目录下，执行 `erl`命令，出现对应的版本号信息则代表安装成功：

```shell
[root@hadoop001 bin]# ./erl
Erlang/OTP 22 [erts-10.4] [source] [64-bit] [smp:2:2] [ds:2:2:10] [async-threads:1]
Eshell V10.4  (abort with ^G)
```

#### > 配置环境变量

```she
vim /etc/profile
```

配置环境变量：

```shell
export ERLANG_HOME=/usr/app/erlang
export PATH=$PATH:$ERLANG_HOME/bin
```

使得配置的环境变量立即生效：

```shell
source /etc/profile
```



### （3）RabbitMQ 安装

#### > 下载并解压

从 RabbitMQ 的 GitHub 仓库进行下载，地址为：https://github.com/rabbitmq/rabbitmq-server/releases/ ：

```shell
# 下载
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.15/rabbitmq-server-generic-unix-3.7.15.tar.xz
# 解压
tar -Jxf rabbitmq-server-generic-unix-3.7.15.tar.xz
```

#### > 配置环境变量

```she
vim /etc/profile
```

配置环境变量：

```shell
export RABBITMQ_HOME=/usr/app/rabbitmq_server-3.7.15
export PATH=$PATH:$RABBITMQ_HOME/sbin
```

使得配置的环境变量立即生效：

```shell
source /etc/profile
```

#### > 启动 RabbitMQ 服务

以后台守护进程的方式启动 RabbitMQ ，命令如下：

```shell
rabbitmq-server start -detached
```

#### > 查看服务状态

```shell
rabbitmqctl status
```



### （4）Web UI界面

#### > 启动 Web UI

想要使用 RabbitMQ 的 Web UI 界面，需要启动管理插件，命令如下：

```shell
rabbitmq-plugins enable rabbitmq_management
```

访问端口为 `15672`。默认的用户名和密码都是 `guest` 。如果你所用浏览器和 RabbitMQ 服务不在同一台主机上，此时应该无法登录，并出现下面的提示 ：

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/02-Java相关技术栈/09-RabbitMq/images/RabbitMQ-访问限制.png)
之所以会出现这个提示，是因为出于安全考虑，RabbitMQ 只允许在本机使用默认的`guest`用户名登录。想要在其他主机上登录，需要使用自定义的账户。

#### > 新增账户

新增用户，用户名和密码都是 root ：

```sh
rabbitmqctl add_user root root
```

赋予用户在默认的名为  `/`  的 Virtual Host 上的所有权限：

```shell
rabbitmqctl set_permissions -p / root '.*' '.*' '.*'
```

设置用户的角色为管理员：

```
rabbitmqctl set_user_tags root administrator
```

#### > 使用新账户登录

登录后可以查看到RabbitMQ 和 Erlang 的版本号，以及对应的账户信息：

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/02-Java相关技术栈/09-RabbitMq/images/rabbitmq-管控台.png)



<div align="center"> <img  src="https://gitee.com/MartinHub/MartinHub-notes/raw/master/images/weixin.png" width="200"/> </div>
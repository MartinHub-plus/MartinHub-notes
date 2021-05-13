## 一、Nginx 编译方式安装

### （1）安装依赖

- **PCRE**：用于支持正则表达式，它主要被 NGINX Core 和 Rewrite 模块所需要：

```shell
# PCRE库的编译需要该组件，所以需要预先安装
$ yum -y install gcc-c++

$ wget https://ftp.pcre.org/pub/pcre/pcre-8.42.tar.gz
$ tar -zxf pcre-8.42.tar.gz
$ cd pcre-8.42
$ ./configure
$ make
$ make install
```

- **zlib** ：用于支持 header 头压缩，它主要被 NGINX Gzip 模块所依赖：

```shell
$ wget http://zlib.net/zlib-1.2.11.tar.gz
$ tar -zxf zlib-1.2.11.tar.gz
$ cd zlib-1.2.11
$ ./configure
$ make
$ make install
```

- **OpenSSL**：用于支持 HTTPS 协议，它主要被 NGINX SSL 模块和其他模块所依赖：

```shell
yum install openssl openssl-devel
```

### （2）安装 Nginx

#### > 下载并解压

下载所需版本的 Nginx 并进行解压，Nginx 版本号的规则为：`主版本号.次版本号.修订号`，其中稳定版的次版本号为偶数，测试版的次版本号为奇数，生产环境下应尽量下载稳定版：

```shell
$ wget https://nginx.org/download/nginx-1.16.1.tar.gz
$ tar -zxvf nginx-1.16.1.tar.gz
```

#### > 编译安装

按需编译 Nginx，可以使用 `--with` 来包含非默认安装的模块，或使用 `--without` 来排除默认安装的模块。关于全部模块的信息，可以使用 ` ./configure  --help` 命令查看或查阅官方文档：https://nginx.org/en/docs/configure.html  ：

```shell
$ cd nginx-1.16.1

$ ./configure   \
--prefix=/usr/app/nginx-1.16.1  \   # 这个路径是准备安装nginx在哪个地方的路径，不是下载nginx源码的路径
--with-pcre=/usr/app/pcre-8.42  \   # 这个路径是pcre的下载路径
--with-zlib=/usr/app/zlib-1.2.11 \ # 这个路径是zlib的下载路径
--with-http_ssl_module \     # 启用HTTPS支持
--with-stream  \         # 启用TCP和UDP代理功能
--with-mail=dynamic         # 启用邮件代理功能

$ make
$ make install
```

#### > 配置环境变量

```shell
$ vim /etc/profile
```

增加如下配置：

```shell
export NGINX_HOME=/usr/app/nginx-1.16.1
export PATH=${NGINX_HOME}/sbin:$PATH
```

使得配置的环境变量立即生效：

```shell
$ source /etc/profile
```

#### > 启动

查看80端口是否打开：`firewall-cmd --query-port=80/tcp`

开启80端口：`firewall-cmd --add-port=80/tcp --permanent`

重启防火墙：`systemctl restart firewalld` 

--permanent   #永久生效，没有此参数重启后失效

## 二、Nginx 基础

### （1）Nginx 简介

#### > 简介

Nginx（engine x）是一个免费的，开源的，高性能的 HTTP 服务器， IMAP/POP3 代理服务器 和 TCP/UDP 代理服务器，通常可以用于进行反向代理和实现基于软件的负载均衡，除此之外，它还具备以下特性：

- Nginx 在设计时遵循模块化的设计方案，可以通过组合模块来扩展实现不同的功能，具备很高的扩展性。
- Nginx 遵循 matser \ worker 架构，主进程负责管理一个或者多个 worker 进程，每个 worker 进程负责处理实际的连接，当某个 worker 进程出错时，主进程会迅速创建一个新的 worker 来继续处理连接请求，从而保证高可用。
- Nginx 在高连接数的情况下仍然可以保持极低的内存占用，从而可以支持高并发量地访问。
- Nginx 支持热部署和配置热加载，并支持在不停机的情况下进行版本升级。
- 有免费的开源版本和商业版本 ( Nginx Plus )，可以按需选择或者进行二次开发。
- 在高并发环境下，Nginx 比其他 WEB 服务器有更快的响应速度。

#### > 正向代理和反向代理

Nginx 能够同时支持正向代理和反向代理，这两种代理模式的区别如下：

- 正向代理发生在客户端，是客户端主动发起的代理。如我们不能直接访问某个服务器，但可以间接通过中间的代理服务器去进行访问，然后将访问结果再返回给我们。
- 反向代理发生在服务端，客户端并不知道发生了代理，示例如下。用户只知道将请求发送给 Nginx，但是并不知道请求被转发了，也不知道被转发给了哪一台应用服务器。实际上对于用户来说，他也没必要知道，因为请求结果都是相同的。

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/04-Linux相关/03-Nginx/images/nginx-plus.png)

### （2）基本命令

Nginx Shell 的基本使用格式如下：

```shell
nginx [-?hvVtTq] [-s signal] [-c filename] [-p prefix] [-g directives]
```

- **-?,-h** ：显示帮助信息；
- **-v**：查看版本号；
- **-V**：查看版本号及配置信息；
- **-t**：检测配置文件是否有语法错误；
- **-q**：静默模式，在检测配置期间除了错误提示外，不输出其他消息；
- **-s signal**：向 Master 进程发送信号，支持以下信号类型：stop ( 立即停止 )，quit ( 优雅停止 )，reload ( 重新加载配置文件 )，reopen (打开一个新的日志文件来继续记录日志) ； 
- **-p prefix** ：指定路径的前缀，默认为安装目录，如 `/usr/app/nginx-1.16.1/` ；
- **-c filename** ：指定配置文件的位置， 默认值为 `conf/nginx.conf`，其实际指向的路径为 `prefix + filename`；
- **-g directives**：从指定的配置文件中设置全局指令。

### （3）配置格式

#### > 基本配置格式

Nginx 的配置由全局配置和局部配置（指令块）共同组成，所有的指令块都遵循相同的配置格式：

```properties
<section>{
    <directive>    <parameters>;
}
```

指令块使用大括号进行划分，大括号内是独立的配置上下文，包含指令和具体的参数，每行指令以分号作为结尾。除此之外，Nginx 的配置中还支持以下操作：

- 支持使用 `include` 语法来引入外部配置文件，从而可以将每个独立的配置拆分到单独的文件中；
- 支持使用 `#` 符号来添加注释；
- 支持使用 `$` 符号来引用变量；
- 部分指令的参数支持使用正则表达式。

#### > 时间和空间单位

Nginx  的配置文件支持以下空间和时间单位：

- **空间单位**：不加任何单位默认就是 bytes，除此之外还支持 k/K，m/M，g/G 等常用单位。
- **空间单位**：支持 ms (毫秒) ，s (秒) ，m (分钟) ，h (小时) ，d (天)，w (星期)，M (月，30天)，y (年，365天) 等常用单位，并支持组合使用，如 `1h 30m` (1小时 30分)，`1y 6M`（1年零6个月）。

#### > 官方配置模板

在安装 Nginx 后，在安装目录的 conf 目录下会有一个官方的配置样例 nginx.conf ，其内容如下：

```properties
# 使用这个参数来配置worker进程的用户和组，如果没有指定组，则默认为指定用户所处的组，默认值为nobody
user  nobody;
# 指定用于处理客户端连接的worker进程的数量，通常设置为CPU的核心数。
# 如果是为IO密集型操作进行负载，可以设置为核心数的1.5 ~ 2倍
worker_processes  1;

# 指定日志的位置和日志级别，日志级别按照由低到高的顺序如下：[debug|info|notice|warn|error|crit]
error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

# 指定记录主进程的ID的文件
pid        logs/nginx.pid;


events {
    # 指定每个工程线程的最大连接数，总的连接数 max_clients = worker_processes * worker_connections
    worker_connections  1024;
}


http {
    # 使用include来引用外部文件
    include       mime.types;
    # 指定默认MIME类型
    default_type  application/octet-stream;

    # 定义日志的输出格式，使用$来进行变量引用
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    # 定义访问日志的存放位置
    access_log  logs/access.log  main;


    # 是否开启系统调用方法sendfile(),开启后可以直接在内核空间完成文件的发送，即零拷贝
    sendfile        on;
    # 是否开启Socket选项,它只有在sendfile启用后才会生效
    tcp_nopush     on;

    # 连接超时时间
    keepalive_timeout  65;

    # 开启文件压缩
    gzip  on;
    gzip_min_length 1k;
    gzip_comp_level 9;
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
    gzip_vary on;
    gzip_disable "MSIE [1-6]\.";


    # 配置nginx服务器(虚拟主机)
    server {
        # 监听端口
        listen       80;
        server_name  localhost;

        # 默认字符集
        charset koi8-r;
        # 配置当前虚拟主机的访问日志的存放位置
        access_log  logs/host.access.log  main;
       
        root         /usr/share/nginx/html/bigdata;
        
        # 虚拟主机对应的映射目录
        location ^~/api/ {
            proxy_pass http://127.0.0.1:8080/api/;
        }
        location ^~/oss-manager/ {
            proxy_pass http://127.0.0.1:8080/oss-manager/;
        }
        location ^~/datahub/ {
            proxy_pass http://127.0.0.1:8080/datahub/;
        }
        location ^~/ambari/ {
            proxy_pass http://127.0.0.1:8080/ambari/;
        }
        location ^~/etl-service/ {
            proxy_pass http://127.0.0.1:8080/etl-service/;
        }

            
        # 错误重定向页面
        # error_page  404              /404.html;
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # 禁止访问根目录下以ht结尾的文件
        location ~ /\.ht {
            deny  all;
        }
    }


    # 支持配置多台虚拟主机
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # 配置Https服务
    server {
        listen       443 ssl;
        server_name  localhost;
        
        # 指定数字证书
        ssl_certificate      cert.pem;
        # 指定密匙
        ssl_certificate_key  cert.key;

        # 设置存储session的缓存类型和大小
        ssl_session_cache    shared:SSL:1m;
        # session缓存时间
        ssl_session_timeout  5m;

        # 返回客户端支持的密码列表
        ssl_ciphers  HIGH:!aNULL:!MD5;
        # 指定在使用SSLv3和TLS协议时，服务器密码应优先于客户端密码
        ssl_prefer_server_ciphers  on;

        location / {
            root   html;
            index  index.html index.htm;
        }
    }
}
```

### （4）部署静态网站

Nginx 通常用作 HTTP 服务器来部署静态资源，其具体的操作步骤如下：

#### > 增加配置

修改 `nginx.conf` ，并在 http 指令块中增加如下配置：

```properties
 server {
    # 监听端口号
    listen 9010;
    # 如果有域名的话，可以在这里进行配置
    server_name _;      
    # 存放静态页面的目录
    root /usr/web;
    # 主页面
    index index.html;
   }
```

在 `/usr/web` 目录下新建一个测试页面 index.html，内容如下：

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport"
        content="width=device-width, user-scalable=no, initial-scale=1.0,
         maximum-scale=1.0, minimum-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Nginx静态资源网站</title>
</head>
<body>
<h1 style="text-align: center">Nginx静态资源网站</h1>
</body>
</html>
```

#### > 检查配置

使用 `-t` 参数来检查配置，出现 successful 则代表配置正确：

```shell
[root@node1 web]# nginx -t -c conf/nginx.conf
nginx: the configuration file /usr/app/nginx-1.16.1/conf/nginx.conf syntax is ok
nginx: configuration file /usr/app/nginx-1.16.1/conf/nginx.conf test is successful
```

#### > 重载配置

启动 Nginx ，如果 Nginx 已经启动，可以使用如下命令重载配置：

```shell
nginx -s reload
```

访问 http://hostname:9010/index.html ，即可查看到静态网站首页。



### （5）实现负载均衡

#### > 部署后台服务

这里我使用 Docker 来部署两个 Tomcat，之后将测试项目 WAR 包分别拷贝到 `/usr/webapps001` 和  `/usr/webapps002`  两个挂载的容器卷下，程序会自动解压并运行，两个项目的端口号分别为 8080 和 8090：

```shell
run -d  -it --privileged=true -v /usr/webapps01:/usr/local/tomcat/webapps \
-p 8080:8080 --name tomcat8080  96c4e536d0eb
```

```shell
run -d  -it  --privileged=true -v /usr/webapps02:/usr/local/tomcat/webapps \
-p 8090:8080 --name tomcat8090  96c4e536d0eb
```

#### > 负载均衡配置

修改 `nginx.conf` ，并在 http 指令块中增加如下配置：

```properties
# 这里指令块的名称可以随意指定，只要和下面的proxy_pass的值相同即可，通常配置为项目名
upstream springboot {
    server 192.168.0.226:8080;
    server 192.168.0.226:8090;
}

server {
    listen 9020;
    location / {
        proxy_pass http://springboot;
    }
}
```

重载配置后，打开浏览器，通过 9020 端口访问项目，此时 Nginx 会以轮询的方式将请求分发到 8080 和 8090 端口上。在测试负载均衡策略的时候，最好将浏览器的缓存功能关闭，避免造成影响。

#### > 负载均衡策略

在上面的配置中，我们没有配置任何负载均衡策略，默认采用的是轮询策略，除此之外，Nginx 还支持以下负载均衡策略：

1. 权重策略 ( Weighted load balancing )

通过为不同的服务分配不同的权重来进行转发，配置示例如下：

```properties
upstream myapp1 {
    server srv1.example.com weight=3;
    server srv2.example.com weight=2;
    server srv3.example.com;
}
```

2. 最少连接策略 ( Least connected load balancing )

将请求转发给连接数最少的服务，配置实例如下：

```properties
upstream myapp1 {
    least_conn;
    server srv1.example.com;
    server srv2.example.com;
    server srv3.example.com;
}
```

3. IP 策略 ( IP Hash load balancing )

通过对请求的 IP 地址进行哈希运算并取模来决定转发对象，配置示例如下：

```properties
upstream myapp1 {
    ip_hash;
    server srv1.example.com;
    server srv2.example.com;
    server srv3.example.com;
}
```

以上是 Nginx 内置的基础的负载均衡策略，如果想要实现其他更加复杂的负载均衡策略，可以通过第三方模块来实现。

#### > 声明备用服务

在上面任何负载均衡策略当中，都可以通过 backup 参数来添加备用服务，示例如下：

```shell
server backup1.example.com:8080   backup;
```

处于备用状态下的服务并不会参与负载均衡，除非所有主服务都已宕机，此时 Nginx 才会将请求转发到备用服务上。



### （6）实现动静分离

#### > 动静分离配置

Nginx 能够支持高并发的访问，并具有静态资源缓存等特性，因此相比于 Tomcat 等动态资源应用服务器，其更加适合于部署静态资源。想要实现动静分离，只需要在 server 指令块中通过正则表达式来划分静态资源，并指定其存放位置，示例如下：

```shell
server {
    listen 9020;
    location / {
        proxy_pass http://springboot;
    }
    # 通过正则来控制所需要分离的静态资源
    location ~ .*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|txt|js|css)$ {
        # 静态资源存放目录
        root /usr/resources/;
    }
}
```

#### > 常见配置异常

1. No such file or directory

第一个常见的问题是找不到静态资源，此时可以查看 logs 目录下的 `error.log` 日志，通常输出如下：

```shell
2019/09/01 17:12:43 [error] 12402#0: *163 open() "/usr/resources/spring-boot-tomcat/css/show.css"
failed (2: No such file or directory), client: 192.168.0.106, server: , 
request: "GET /spring-boot-tomcat/css/show.css HTTP/1.1", host: "192.168.0.226:9020",
referrer: "http://192.168.0.226:9020/spring-boot-tomcat/index"
```

出现这个问题，是因为 Nginx 要求静态资源的请求路径必须和原有请求路径完全相同，这里我的项目在 Tomcat 中解压后的项目名为 `pring-boot-tomcat`，以 show.css 文件为例，其正确的存储路径应该为：

```shell
/usr/resources/spring-boot-tomcat/css/show.css
```

即： 静态资源根目录 + 项目名 + 原有路径，通常我们在创建目录时会忽略掉项目名这一层级，从而导致异常。

2. Permission denied

路径正确后，另外一个常见的问题是权限不足，错误日志如下。此时需要保证配置文件中的 user 用户必须具有静态资源所处目录的访问权限，或者在创建静态资源目录时，直接使用 user 配置的用户来创建：

```shell
2019/09/01 17:15:14 [error] 12402#0: *170 open() "/usr/resources/spring-boot-tomcat/css/show.css" 
failed (13: Permission denied), client: 192.168.0.106, server: ,
request: "GET /spring-boot-tomcat/css/show.css HTTP/1.1", host: "192.168.0.226:9020", 
referrer: "http://192.168.0.226:9020/spring-boot-tomcat/index"
```





### 参考资料

官方文档：[nginx documentation](http://nginx.org/en/docs/) ，[Using nginx as HTTP load balancer](http://nginx.org/en/docs/http/load_balancing.html)





<div align="center"> <img  src="https://gitee.com/MartinHub/MartinHub-notes/raw/master/images/weixin.png" width="200"/> </div>
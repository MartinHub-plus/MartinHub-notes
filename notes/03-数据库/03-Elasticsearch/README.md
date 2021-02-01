## 一、ElasticSearch 和 Kibana 单机环境搭建

### （1）ElasticSearch 安装

#### > 下载并解压

Elastic 所有软件的下载地址均为 ：https://www.elastic.co/cn/downloads/past-releases ，下载相同版本的 ElasticSearch 和 Kibana，下载后进行解压：

```shell
tar -zxvf elasticsearch-7.2.0-linux-x86_64.tar.gz -C /usr/app/
tar -zxvf kibana-7.2.0-linux-x86_64.tar.gz -C /usr/app/
```

#### > 修改软件配置

修改安装目录下的 `config/elasticsearch.yml` 文件，修改内容如下：

```shell
# 当前节点的名称
node.name: node-1
# 修改绑定地址，默认为本机地址，此时只能在本机访问ElasticSearch服务，想要所有主机都能访问，则修改为0.0.0.0
network.host: 0.0.0.0
http.port: 9200
cluster.initial_master_nodes: ["node-1"]
```

#### > 修改服务器配置

如果你采用默认的服务器配置进行启动，通常会抛出下面两个异常：

```shell
ERROR: [2] bootstrap checks failed
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

此时需要对服务器配置进行两项更改：

- **sysctl.conf** 

编辑 `/etc/sysctl.conf` 文件，增加如下配置： 

```shell
# 限制一个进程可以拥有的VMA(虚拟内存区域)的数量
vm.max_map_count=262144
```

修改完成后，如果想要在不重启的情况下使得该配置生效，则需要执行以下命令：

```shell
/sbin/sysctl -p
```

- **limits.conf** 

编辑 `/etc/security/limits.conf` 文件，增加如下配置：

```
* soft nofile 65536
* hard nofile 65536
root soft nofile 65536
root hard nofile 65536
```

修改配置完成重启 shell 客户端即可生效，想要验证是否生效，可以使用命令 `ulimit -a` 查看输出中的 `open files` 的值是否为 65536 。

#### > 启动服务

进入安装目录的 `bin` 目录下，执行以下命令启动服务。这里为了观察效果使用前台方式启动，如果想要以后台进程的方式启动，则需要在后面加上 `-d` 参数：

```she
./elasticsearch
```

需要特别注意的是，处于安全的考虑，Elasticsearch 不允许使用 root 账户启动服务，所以启动时需要切换到其他用户。同时启动用户必须拥有 Elasticsearch 目录的访问权限，可以先使用 `chown` 命令授权后再使用 `su` 命令切换到对应用户，示例如下： 

```shell
useradd heibaiying
chown -R heibaiying:heibaiying /usr/app/elasticsearch-7.2.0/
```

#### > 启动验证

想要验证是否启动成功，可以使用 `jps` 命令查看 `Elasticsearch` 进程是否启动，也可以访问`9200`端口，出现如下页面则代表启动成功：

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/03-数据库/03-Elasticsearch/images/elk-web-ui.png)

### （2）Kibana 安装

#### > 修改配置

修改安装目录下的 `config/kibana.yml` 文件，修改内容如下：

```shell
server.port: 5601
server.host: 0.0.0.0
```

#### > 启动服务

同样 Kibana 不允许使用 root 账户启动服务，所以启动时需要切换到其他用户。同时启动用户必须拥有 Kibana 目录的访问权限，可以先使用 `chown` 命令授权后再使用 `su` 命令切换到对应用户，示例如下： 

```shell
chown -R heibaiying:heibaiying /usr/app/kibana-7.2.0-linux-x86_64/
```

切换用户，并进入到安装目录的 `bin` 目录下，启动服务：

```shell
# 切换用户
su - heibaiying
# 启动服务
./kibana
```

需要说明的是这里 kibana 启动时候没有 `-d` 这个参数，想要后台启动，则可以使用以下命令：

```shell
nohup ./kibana  &
```

#### > 访问页面

kibana Web UI 的访问端口号为`5601`，出现以下页面则代表启动成功：

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/03-数据库/03-Elasticsearch/images/kibana-web-ui.png)

#### > 界面汉化

kibana 7.0 的界面支持中文显示，可以在 `conf/kibana.yml` 中增加如下选项来进行配置：

```shell
i18n.locale: "zh-CN"
```


## 二、Elasticsearch 7.x 基本操作

### （1）索引管理

#### > 新建索引

使用指定配置创建索引，这里指定主分片的数量为 3； 副本系数为 2，即每个分片两个副本；默认情况下主分片数量和副本系数都是 1。需要注意的是创建索引时，索引名称只能是小写，长度不能超过 255 个字符，同时尽量不要包含特殊字符，不能以 `-`，`_`，`+` 等字符开头。

```json
PUT weibo
{
    "settings" : {
        "number_of_shards" : 3, 
        "number_of_replicas" : 2 
    }
}
```

#### > 修改配置

```json
PUT weibo/_settings
{
  "number_of_replicas": 1
}
```

ES 支持修改索引的副本系数，但不支持随意修改索引的主分片数，这与 ES 分片的路由机制有关，ES 使用以下公式来决定每条数据存储在哪个分片上：

```java
shard = hash(routing) % number_of_primary_shards
```

routing 是一个任意的字符串，默认是 `_id` ，即使用 `_id` 进行分片计算，当然也支持自定义分片键。ES 对其进行哈希运算然后按 `number_of_primary_shards` 取余后计算出存储分片的序号。基于这个原因，所以 ES 不允许对 `number_of_primary_shards` 进行修改，因为这会导致已有数据存储位置的失效。

#### > 查看与删除

查看索引、查看索引配置、删除索引的语法分别如下：

```shell
# 查看索引信息
GET weibo

# 查看索引配置信息
GET weibo/_settings

# 删除索引
DELETE weibo
```

#### > 打开与关闭

ES 中的索引支持打开和关闭操作，索引关闭后就无法进行读写操作，因此其占用的系统资源也会随之减少。索引打开和关闭的语法为：

```shell
# 关闭索引
POST weibo/_close

# 打开索引
POST weibo/_open
```

ES 支持一次关闭多个索引，多个索引名之间需要使用逗号分隔，同时也支持使用通配符和 `_all` 关键字，示例如下：

```shell
# 关闭多个索引，ignore_unavailable=true代表如果其中某个索引不存在，则忽略该索引
POST weibo1,weibo2,weibo3/_close?ignore_unavailable=true

# 支持通配符
POST weibo*/_close

# 关闭集群中所有索引
POST _all/_close
```

### （2）单文档操作

#### >  新建文档

```json
PUT weibo/_doc/1
{
  "user" : "kimchy",
  "post_date" : "2009-11-15T14:12:12",
  "message" : "trying out Elasticsearch"
}
```

这里需要注意的是在 7.x 版本后  ES 已经不推荐使用文档类型的概念，所以这里的 `_doc` 表示端点名称而不是文档类型。输出如下：

```json
{
  "_index" : "weibo",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}

```

1. **_id**

在输出中可以看到 `_id` 的值为 1，这是在创建时指定的。如果创建时没有指定，则由 ES 自动生成。

2. **_version**

`_version` 为文档版本号，基于它可以实现乐观锁的效果，需要配合 `version_type` 使用，`version_type` 有以下可选值：

- **interna** ：当给定的版本号与文档版本号相同时候，才执行对应的操作；
- **external 或 external_gt** ：如果给定版本号大于存储文档的版本或者原文档不存在时，才执行对应的操作，给定的版本号将用作新的版本；
- **external_gte** ：和上一条类似，等价于 `gt + equal` ，即给定版本号大于或等于存储文档的版本或者原文档不存在时，才执行对应的操作。

使用示例：`PUT weibo/_doc/1?version=2&version_type=external { ... }`

3. **_shards**

输出结果中的 `_shards` 节点下有三个参数，其含义分别如下：

- **total** ：操作涉及到的分片总数，这里由于创建 `weibo` 索引时指定主分片数量和副本系数都是 1，所以 只有 1 个 primary 分片和 1 个 replica 分片，故总数为 2； 
- **successful** ：这里我采用的是单节点的 ES , replica 分片实际上是不存在的。因为按照 ES 的规则，primary 分片及其对应的 replica 分片不能处于同一台主机上，因为处于同一台主机上时无法达到容错的效果。所以这里只有 primary 分片写入数据成功，故值为 1；
- **failed** ：执行复制操作失败的 replica 分片的数量，这里由于 replica 分片本生就不存在所以值为 0 。

**4. routing** 

在上面我们提到，ES 的分片路由规则默认进行哈希的对象是 `_id`，如果你想指定使用其他字段，则可以使用 `routing` 参数进行指定，示例如下：

```json
POST weibo/_doc?routing=kimchy
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

#### > 查询文档

```shell
GET weibo/_doc/1
```

文档的实际内容位于查询结果的 `_source` 节点下，输出如下：

```json
{
  "_index" : "weibo",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 3,
  "_seq_no" : 2,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
  }
}
```

默认情况下会返回文档全部字段，如果你只想返回部分字段或者排除部分字段，则可以使用 `_source_includes` 关键字或 `_source_excludes` 关键字：

```shell
GET weibo/_doc/1?_source_includes=user
GET weibo/_doc/1?_source_excludes=message
```

如果你不想返回文档内容，即不需要 `_source` 节点，则可以使用以下语法：

```shell
GET weibo/_doc/1?_source=false
```

如果你只想返回文档内容，即只需要 `_source` 节点，则可以使用以下语法：

```shell
GET weibo/_source/1
```

此时依然支持使用 `_source_includes` 和 `_source_excludes` 来包含和排除部分字段：

```
GET weibo/_source/1?_source_includes=user
GET weibo/_source/1?_source_excludes=message
```

#### > 更新文档

新增相同 `_id` 的文档等价于执行更新操作，此时会用新的文档内容完全替换已有的文档内容，示例如下：

```json
POST weibo/_doc/1
{
  "user" : "Tom",
  "post_date" : "2019-11-15T14:12:12",
  "message" : "trying out Hadoop"
}
```

如果想要部分更新，可以使用 `_update` 语法：

```json
POST weibo/_update/1
{
   "doc":{ 
      "message":"trying out Spark"
   }
}
```

如果更新操作需要依赖原值，这里假设要更新的用户名需要由之前的用户名拼接而成，此时语法如下。更新完成后，新的用户名为 `Tom Cat` 。这里的 painless 是 ES 内置的一种脚本语言，params 是参数的集合，ctx 是上下文执行对象，其 `_source` 指向原文档的内容：

```json
POST  weibo/_update/1/
{
    "script":{
        "source":"ctx._source.user+=params.user",
        "lang":"painless",
        "params":{
            "user" : " Cat"
        }
    }
}
```

#### > 删除文档

对文档执行的每个写入操作（包括删除）都会导致其版本号递增。删除文档时，可以通过指定版本号来确保文档在删除期间没有被其它用户更改；被删除的文档版本在删除后的短时间内仍使用，以便更好地支持并发操作。该时间长度由 `index.gc_deletes` 索引设置，默认为 60 秒。删除示例如下：

```shell
DELETE /weibo/_doc/1?version=2&version_type=interna
```

### （3）多文档操作

#### > 批量获取

想要查询多个文档，可以使用 `_mget` 语法，示例如下：

```json
GET /weibo/_mget
{
    "docs" : [
        {
            "_id" : "1"
        },
        {
            "_id" : "2"
        }
    ]
}
```

上面的查询语句也可以简写如下：

```json
GET /weibo/_mget
{
    "ids" : ["1", "2"]
}
```

和普通查询一样，如果想要针对不同的 `_id` 选择不同的字段，可以使用以下语法：

```json
GET /weibo/_mget
{
    "docs" : [
        {
            "_id" : "1",
            "_source":["user","message"]
        },
        {
            "_id" : "2",
            "_source" : {
                "include": ["message"],
                "exclude": ["user"]
            }
        }
    ]
}
```

如果想要所有的文档都返回相同的字段，可以使用以下语法：

```json
GET /weibo/_mget?_source_includes=user,message
{
    "ids" : ["1", "2"]
}
```

#### > 批量更新

```json
POST _bulk
{ "update" : {"_id" : "1", "_index" : "weibo", "retry_on_conflict" : 3} }
{ "doc" : {"user" : "heibai"} }
{ "update" : {"_id" : "2", "_index" : "weibo", "_source" : true} }
{ "script":{"source":"ctx._source.user+=params.user","lang":"painless","params":{"user" : " Cat"}}}
```

上面是一个批量更新的示例，其抽象语法格式如下。需要注意的是这里语法是非常严格的：一个更新操作必须由两行 Json 组成，第一行 Json 用于指明操作，文档位置和其他可选配置，第二行 Json 用于存放数据或指明更新的方式：

```txt
POST _bulk
{"操作" : {"文档位置","可选操作"}}
{"数据"}
{"操作" : {"文档位置","可选操作"}}
{"数据"}
{"操作" : {"文档位置","可选操作"}}
{"数据"}
....
```

另外一个需要注意的是每行 Json 都必须遵循单行模式，不能是多行模式，即如下的语法是错误的：

```json
{
    "script": {
        "source": "ctx._source.user+=params.user",
        "lang": "painless",
        "params": {
            "user": " Cat"
        }
    }
}
```

之所以这样设计是因为如果允许多行模式，其解析就会比较繁琐且耗费性能，假设我们一次批量执行上万个更新，则用于描述操作的 Json 文件就会非常大，此时程序需要将其拷贝到内存中先进行解析，这个操作既浪费内存又浪费计算资源。而采用单行模式就能避免这个问题，程序只需要逐行读取记录，每读取两行则必然就是一个完整的操作，此时只需要将其发送到对应分片节点上执行即可。



<div align="center"> <img  src="https://gitee.com/MartinHub/MartinHub-notes/raw/master/images/weixin.png" width="200"/> </div>
## 一、JanusGraph的搭建

### （1）搭建准备

本文主要记录 JanusGraph 图数据库的部署，需要另行搭建 HBase 和 ElasticSearch，本文以本地已经部署好 HBase 和 ElasticSearch 为例。

- HBase 地址: `127.0.0.1:2181`
- ElasticSearch 地址: `127.0.0.1:9200`

### （2）文件准备

> 项目地址: [JanusGraph/janusgraph](https://github.com/JanusGraph/janusgraph)

在 [Github release 页面](https://github.com/JanusGraph/janusgraph/releases)上下载我们所需要的文件，形如 `janusgraph-{VERSION}-hadoop2.zip`，在这里我们下载最新版 `0.3.2` 版本: [`janusgraph-0.3.2-hadoop2.zip`](https://github.com/JanusGraph/janusgraph/releases/download/v0.3.2/janusgraph-0.3.2-hadoop2.zip)，并解压出来。

```shell
cd ~
wget https://github.com/JanusGraph/janusgraph/releases/download/v0.3.2/janusgraph-0.3.2-hadoop2.zip -cO janusgraph-0.3.2-hadoop2.zip
unzip janusgraph-0.3.2-hadoop2.zip
mv janusgraph-0.3.2-hadoop2 janusgraph
cd janusgraph
```

接下来的操作大部分在此文件夹内进行，即 `~/janusgraph` 文件夹内。

### （3）调整 Gremlin Server 配置

通过 `vi conf/gremlin-server/gremlin-server.yaml` 打开，有以下部分配置需要改动。

```shell
# 监听地址，默认对外开放，若只允许本地访问则更改为 127.0.0.1
host: 0.0.0.0
# 监听端口，有需要可以更改
port: 8182
# 服务类型，可选以下内容
# - WebSocketChannelizer 提供WebSocket服务
# - HttpChannelizer 提供Http服务
# - WsAndHttpChannelizer 推荐，同时提供WebSocket和Http服务，从0.2.0版本开始支持
channelizer: org.apache.tinkerpop.gremlin.server.channel.WsAndHttpChannelizer
graphs: {
    # 所要用到的配置文件路径，可自定义
    graph: conf/gremlin-server/janusgraph.properties,
  }
```

- 完整配置

```properties
host: 0.0.0.0
port: 8182
scriptEvaluationTimeout: 30000
channelizer: org.apache.tinkerpop.gremlin.server.channel.WsAndHttpChannelizer
graphs: { graph: conf/gremlin-server/janusgraph.properties }
scriptEngines:
  {
    gremlin-groovy:
      {
        plugins:
          {
            org.janusgraph.graphdb.tinkerpop.plugin.JanusGraphGremlinPlugin: {},
            ? org.apache.tinkerpop.gremlin.server.jsr223.GremlinServerGremlinPlugin
            : {},
            ? org.apache.tinkerpop.gremlin.tinkergraph.jsr223.TinkerGraphGremlinPlugin
            : {},
            org.apache.tinkerpop.gremlin.jsr223.ImportGremlinPlugin:
              {
                classImports: [java.lang.Math],
                methodImports: [java.lang.Math#*],
              },
            org.apache.tinkerpop.gremlin.jsr223.ScriptFileGremlinPlugin:
              { files: [scripts/empty-sample.groovy] },
          },
      },
  }
serializers:
  - {
      className: org.apache.tinkerpop.gremlin.driver.ser.GryoMessageSerializerV3d0,
      config:
        {
          ioRegistries: [org.janusgraph.graphdb.tinkerpop.JanusGraphIoRegistry],
        },
    }
  - {
      className: org.apache.tinkerpop.gremlin.driver.ser.GryoMessageSerializerV3d0,
      config: { serializeResultToString: true },
    }
  - {
      className: org.apache.tinkerpop.gremlin.driver.ser.GraphSONMessageSerializerV3d0,
      config:
        {
          ioRegistries: [org.janusgraph.graphdb.tinkerpop.JanusGraphIoRegistry],
        },
    }
  # Older serialization versions for backwards compatibility:
  - {
      className: org.apache.tinkerpop.gremlin.driver.ser.GryoMessageSerializerV1d0,
      config:
        {
          ioRegistries: [org.janusgraph.graphdb.tinkerpop.JanusGraphIoRegistry],
        },
    }
  - {
      className: org.apache.tinkerpop.gremlin.driver.ser.GryoLiteMessageSerializerV1d0,
      config:
        {
          ioRegistries: [org.janusgraph.graphdb.tinkerpop.JanusGraphIoRegistry],
        },
    }
  - {
      className: org.apache.tinkerpop.gremlin.driver.ser.GryoMessageSerializerV1d0,
      config: { serializeResultToString: true },
    }
  - {
      className: org.apache.tinkerpop.gremlin.driver.ser.GraphSONMessageSerializerGremlinV2d0,
      config:
        {
          ioRegistries: [org.janusgraph.graphdb.tinkerpop.JanusGraphIoRegistry],
        },
    }
  - {
      className: org.apache.tinkerpop.gremlin.driver.ser.GraphSONMessageSerializerGremlinV1d0,
      config:
        {
          ioRegistries:
            [org.janusgraph.graphdb.tinkerpop.JanusGraphIoRegistryV1d0],
        },
    }
  - {
      className: org.apache.tinkerpop.gremlin.driver.ser.GraphSONMessageSerializerV1d0,
      config:
        {
          ioRegistries:
            [org.janusgraph.graphdb.tinkerpop.JanusGraphIoRegistryV1d0],
        },
    }
processors:
  - {
      className: org.apache.tinkerpop.gremlin.server.op.session.SessionOpProcessor,
      config: { sessionTimeout: 28800000 },
    }
  - {
      className: org.apache.tinkerpop.gremlin.server.op.traversal.TraversalOpProcessor,
      config: { cacheExpirationTime: 600000, cacheMaxSize: 1000 },
    }
metrics:
  {
    consoleReporter: { enabled: true, interval: 180000 },
    csvReporter:
      {
        enabled: true,
        interval: 180000,
        fileName: /tmp/gremlin-server-metrics.csv,
      },
    jmxReporter: { enabled: true },
    slf4jReporter: { enabled: true, interval: 180000 },
    gangliaReporter:
      { enabled: false, interval: 180000, addressingMode: MULTICAST },
    graphiteReporter: { enabled: false, interval: 180000 },
  }
maxInitialLineLength: 4096
maxHeaderSize: 8192
maxChunkSize: 8192
maxContentLength: 65536
maxAccumulationBufferComponents: 1024
resultIterationBatchSize: 64
writeBufferLowWaterMark: 32768
writeBufferHighWaterMark: 65536
```

### （5）调整 properties 文件

通过 `vi conf/gremlin-server/janusgraph.properties` 打开 properties 文件，加入以下内容。

```properties
gremlin.graph = org.janusgraph.core.JanusGraphFactory
storage.backend = hbase
# 可换成远程 HBase 所在 IP
storage.hostname = 127.0.0.1
# 若采用 HBase 集群，使用下面的写法，采用','分隔各个 IP
# storage.hostname = 192.168.0.1,192.168.0.2
storage.port = 2181
cache.db-cache = true
cache.db-cache-clean-wait = 20
cache.db-cache-time = 180000
cache.db-cache-size = 0.5
index.search.backend = elasticsearch
# 可换成远程 ElasticSearch 所在地址
index.search.hostname = 127.0.0.1:9200
# 若采用 ElasticSearch 集群，使用下面的写法，采用','分隔各个 IP，默认端口可省略
index.search.hostname = 192.168.0.1,192.168.0.2:9200,192.168.0.3
```

### （6）临时启用服务

运行如下命令启动服务。

```shell
bin/gremlin-server.sh conf/gremlin-server/gremlin-server.yaml
```

由于使用了默认配置文件，则可以省略配置文件路径。

```powershell
bin/gremlin-server.sh
```

当屏幕上出现如下内容时，则代表服务已经开启完成。

```shell
[gremlin-server-boss-1] INFO  org.apache.tinkerpop.gremlin.server.GremlinServer  - Channel started at port 8182.
```

### （7）守护进程

- 配置

在此以 systemd 为例进行配置，可以参考一下，注意替换文件路径。

```shell
sudo vi /etc/systemd/system/janusgraph.service
```

- 输入以下内容并保存:

```properties
[Unit]
Description=JanusGraph Server

[Service]
# Environment="PATH=yourpath"
ExecStart=/root/janusgraph/bin/gremlin-server.sh /root/janusgraph/conf/gremlin-server/gremlin-server.yaml
ExecReload=/bin/kill -HUP $MAINPID
Type=simple
User=root
Group=root
Restart=always

[Install]
WantedBy=multi-user.target

```

可能会遇到错误，显示 `java: command not found`，请把上面的备注取消掉，替换上自己的系统环境变量。

### （8）常用命令

- 启用服务: `sudo service janusgraph start`
- 关闭服务: `sudo service janusgraph stop`
- 重启服务: `sudo service janusgraph restart`
- 开机启动: `sudo systemctl enable janusgraph`
- 取消开机启动: `sudo systemctl disable janusgraph`
- 查看日志: `sudo journalctl -u janusgraph --since tody`

### （9）测试

- **测试 WebSocket** 

运行 `bin/gremlin.sh`。

```
[admin@localhost janusgraph]$ bin/gremlin.sh

         \,,,/
         (o o)
-----oOOo-(3)-oOOo-----
plugin activated: janusgraph.imports
plugin activated: tinkerpop.server
plugin activated: tinkerpop.utilities
plugin activated: tinkerpop.hadoop
plugin activated: tinkerpop.spark
plugin activated: tinkerpop.tinkergraph
gremlin> :remote connect tinkerpop.server conf/remote.yaml
==>Configured localhost/127.0.0.1:8182
gremlin> :> g.V().count()
==>0
gremlin>

```

如能正常响应，则表示部署成功。

- **测试 Http** 

运行如下命令测试 http 能否正常响应。

```
curl -XPOST -Hcontent-type:application/json -d '{"gremlin":"g.V().count()"}' http://localhost:8182

```

应有类似如下返回内容，则为正常。

```
{
  "requestId": "47608dd1-275d-4708-acf7-fa1e6355328b",
  "status": {
    "message": "",
    "code": 200,
    "attributes": { "@type": "g:Map", "@value": [] }
  },
  "result": {
    "data": {
      "@type": "g:List",
      "@value": [{ "@type": "g:Int64", "@value": 0 }]
    },
    "meta": { "@type": "g:Map", "@value": [] }
  }
}

```

### （10）可能遇到的坑

1. 本地是否安装 java
2. 配置文件路径是否正确
3. 远程 hbase 和 es 端口防火墙是否开放
4. 远程 hbase 和 es 是否监听的是本地地址

<div align="center"> <img  src="https://gitee.com/MartinHub/MartinHub-notes/raw/master/images/weixin.png" width="200"/> </div>
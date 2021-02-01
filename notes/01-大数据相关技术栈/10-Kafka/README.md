![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/10-Kafka/images/kafka.png)



## 一、Kafka简介

### （1）简介

&emsp;ApacheKafka 是一个分布式的流处理平台。它具有以下特点：

- 支持消息的发布和订阅，类似于 RabbtMQ、ActiveMQ 等消息队列；
- 支持数据实时处理；
- 能保证消息的可靠性投递；
- 支持消息的持久化存储，并通过多副本分布式的存储方案来保证消息的容错；
- 高吞吐率，单 Broker 可以轻松处理数千个分区以及每秒百万级的消息量。

### （2）基本概念

#### > Messages And Batches

&emsp;Kafka 的基本数据单元被称为 message(消息)，为减少网络开销，提高效率，多个消息会被放入同一批次 (Batch) 中后再写入。

#### > Topics And Partitions

&emsp;Kafka 的消息通过 Topics(主题) 进行分类，一个主题可以被分为若干个 Partitions(分区)，一个分区就是一个提交日志 (commit log)。消息以追加的方式写入分区，然后以先入先出的顺序读取。Kafka 通过分区来实现数据的冗余和伸缩性，分区可以分布在不同的服务器上，这意味着一个 Topic 可以横跨多个服务器，以提供比单个服务器更强大的性能。

&emsp;由于一个 Topic 包含多个分区，因此无法在整个 Topic 范围内保证消息的顺序性，但可以保证消息在单个分区内的顺序性。

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/10-Kafka/images/kafka-topic.png) 

#### > Producers And Consumers

- **生产者** 

&emsp;生产者负责创建消息。一般情况下，生产者在把消息均衡地分布到在主题的所有分区上，而并不关心消息会被写到哪个分区。如果我们想要把消息写到指定的分区，可以通过自定义分区器来实现。

- **消费者** 

&emsp;消费者是消费者群组的一部分，消费者负责消费消息。消费者可以订阅一个或者多个主题，并按照消息生成的顺序来读取它们。消费者通过检查消息的偏移量 (offset) 来区分读取过的消息。偏移量是一个不断递增的数值，在创建消息时，Kafka 会把它添加到其中，在给定的分区里，每个消息的偏移量都是唯一的。消费者把每个分区最后读取的偏移量保存在 Zookeeper 或 Kafka 上，如果消费者关闭或者重启，它还可以重新获取该偏移量，以保证读取状态不会丢失。

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/10-Kafka/images/kafka-producer-consumer.png) 

&emsp;一个分区只能被同一个消费者群组里面的一个消费者读取，但可以被不同消费者群组中所组成的多个消费者共同读取。多个消费者群组中消费者共同读取同一个主题时，彼此之间互不影响。

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/10-Kafka/images/kafka消费者.png) 

#### > Brokers And Clusters

&emsp;一个独立的 Kafka 服务器被称为 Broker。Broker 接收来自生产者的消息，为消息设置偏移量，并提交消息到磁盘保存。Broker 为消费者提供服务，对读取分区的请求做出响应，返回已经提交到磁盘的消息。

&emsp;Broker 是集群 (Cluster) 的组成部分。每一个集群都会选举出一个 Broker 作为集群控制器 (Controller)，集群控制器负责管理工作，包括将分区分配给 Broker 和监控 Broker。

&emsp;在集群中，一个分区 (Partition) 从属一个 Broker，该 Broker 被称为分区的首领 (Leader)。一个分区可以分配给多个 Brokers，这个时候会发生分区复制。这种复制机制为分区提供了消息冗余，如果有一个 Broker 失效，其他 Broker 可以接管领导权。

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/10-Kafka/images/kafka-cluster.png) 

## 二、Kafka生产者详解

### （1）生产者发送消息的过程

&emsp;首先介绍一下 Kafka 生产者发送消息的过程：

- Kafka 会将发送消息包装为 ProducerRecord 对象， ProducerRecord 对象包含了目标主题和要发送的内容，同时还可以指定键和分区。在发送 ProducerRecord 对象前，生产者会先把键和值对象序列化成字节数组，这样它们才能够在网络上传输。
- 接下来，数据被传给分区器。如果之前已经在 ProducerRecord 对象里指定了分区，那么分区器就不会再做任何事情。如果没有指定分区 ，那么分区器会根据 ProducerRecord 对象的键来选择一个分区，紧接着，这条记录被添加到一个记录批次里，这个批次里的所有消息会被发送到相同的主题和分区上。有一个独立的线程负责把这些记录批次发送到相应的 broker 上。
- 服务器在收到这些消息时会返回一个响应。如果消息成功写入 Kafka，就返回一个 RecordMetaData 对象，它包含了主题和分区信息，以及记录在分区里的偏移量。如果写入失败，则会返回一个错误。生产者在收到错误之后会尝试重新发送消息，如果达到指定的重试次数后还没有成功，则直接抛出异常，不再重试。

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/10-Kafka/images/kafka-send-messgaes.png)  

### （2）创建生产者

#### > 项目依赖

&emsp;本项目采用 Maven 构建，想要调用 Kafka 生产者 API，需要导入 `kafka-clients` 依赖，如下：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.2.0</version>
</dependency>
```

#### > 创建生产者

&emsp;创建 Kafka 生产者时，以下三个属性是必须指定的：

- **bootstrap.servers** ：指定 broker 的地址清单，清单里不需要包含所有的 broker 地址，生产者会从给定的 broker 里查找 broker 的信息。不过建议至少要提供两个 broker 的信息作为容错；
- **key.serializer** ：指定键的序列化器；
- **value.serializer** ：指定值的序列化器。

&emsp;创建的示例代码如下：

```scala
public class SimpleProducer {

    public static void main(String[] args) {

        String topicName = "Hello-Kafka";

        Properties props = new Properties();
        props.put("bootstrap.servers", "hadoop001:9092");
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        /*创建生产者*/
        Producer<String, String> producer = new KafkaProducer<>(props);

        for (int i = 0; i < 10; i++) {
            ProducerRecord<String, String> record = new ProducerRecord<>(topicName, "hello" + i, 
                                                                         "world" + i);
            /* 发送消息*/
            producer.send(record);
        }
        /*关闭生产者*/
        producer.close();
    }
}
```

#### > 测试

- **启动Kakfa**

&emsp;Kafka 的运行依赖于 zookeeper，需要预先启动，可以启动 Kafka 内置的 zookeeper，也可以启动自己安装的：

```shell
# zookeeper启动命令
bin/zkServer.sh start

# 内置zookeeper启动命令
bin/zookeeper-server-start.sh config/zookeeper.properties
```

&emsp;启动单节点 kafka 用于测试：

```shell
# bin/kafka-server-start.sh config/server.properties
```

- **创建topic**

```shell
# 创建用于测试主题
bin/kafka-topics.sh --create \
                    --bootstrap-server hadoop001:9092 \
                     --replication-factor 1 --partitions 1 \
                     --topic Hello-Kafka

# 查看所有主题
 bin/kafka-topics.sh --list --bootstrap-server hadoop001:9092
```

- **启动消费者**

 启动一个控制台消费者用于观察写入情况，启动命令如下：

```shell
# bin/kafka-console-consumer.sh --bootstrap-server hadoop001:9092 --topic Hello-Kafka --from-beginning
```

- **运行项目** 

&emsp;此时可以看到消费者控制台，输出如下，这里 `kafka-console-consumer` 只会打印出值信息，不会打印出键信息。

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/10-Kafka/images/kafka-simple-producer.png) 



#### > 可能出现的问题

&emsp;在这里可能出现的一个问题是：生产者程序在启动后，一直处于等待状态。这通常出现在你使用默认配置启动 Kafka 的情况下，此时需要对 `server.properties` 文件中的 `listeners` 配置进行更改：

```shell
# hadoop001 为我启动kafka服务的主机名，你可以换成自己的主机名或者ip地址
listeners=PLAINTEXT://hadoop001:9092
```



### （3）发送消息

&emsp;上面的示例程序调用了 `send` 方法发送消息后没有做任何操作，在这种情况下，我们没有办法知道消息发送的结果。想要知道消息发送的结果，可以使用同步发送或者异步发送来实现。

#### > 同步发送

&emsp;在调用 `send` 方法后可以接着调用 `get()` 方法，`send` 方法的返回值是一个 Future\<RecordMetadata>对象，RecordMetadata 里面包含了发送消息的主题、分区、偏移量等信息。改写后的代码如下：

```scala
for (int i = 0; i < 10; i++) {
    try {
        ProducerRecord<String, String> record = new ProducerRecord<>(topicName, "k" + i, "world" + i);
        /*同步发送消息*/
        RecordMetadata metadata = producer.send(record).get();
        System.out.printf("topic=%s, partition=%d, offset=%s \n",
                metadata.topic(), metadata.partition(), metadata.offset());
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }
}
```

&emsp;此时得到的输出如下：偏移量和调用次数有关，所有记录都分配到了 0 分区，这是因为在创建 `Hello-Kafka` 主题时候，使用 `--partitions` 指定其分区数为 1，即只有一个分区。

```shell
topic=Hello-Kafka, partition=0, offset=40 
topic=Hello-Kafka, partition=0, offset=41 
topic=Hello-Kafka, partition=0, offset=42 
topic=Hello-Kafka, partition=0, offset=43 
topic=Hello-Kafka, partition=0, offset=44 
topic=Hello-Kafka, partition=0, offset=45 
topic=Hello-Kafka, partition=0, offset=46 
topic=Hello-Kafka, partition=0, offset=47 
topic=Hello-Kafka, partition=0, offset=48 
topic=Hello-Kafka, partition=0, offset=49 
```

#### > 异步发送

&emsp;通常我们并不关心发送成功的情况，更多关注的是失败的情况，因此 Kafka 提供了异步发送和回调函数。 代码如下：

```scala
for (int i = 0; i < 10; i++) {
    ProducerRecord<String, String> record = new ProducerRecord<>(topicName, "k" + i, "world" + i);
    /*异步发送消息，并监听回调*/
    producer.send(record, new Callback() {
        @Override
        public void onCompletion(RecordMetadata metadata, Exception exception) {
            if (exception != null) {
                System.out.println("进行异常处理");
            } else {
                System.out.printf("topic=%s, partition=%d, offset=%s \n",
                        metadata.topic(), metadata.partition(), metadata.offset());
            }
        }
    });
}
```



### （4）自定义分区器

&emsp;Kafka 有着默认的分区机制：

- 如果键值为 null， 则使用轮询 (Round Robin) 算法将消息均衡地分布到各个分区上；
- 如果键值不为 null，那么 Kafka 会使用内置的散列算法对键进行散列，然后分布到各个分区上。

&emsp;某些情况下，你可能有着自己的分区需求，这时候可以采用自定义分区器实现。这里给出一个自定义分区器的示例：

#### > 自定义分区器

```java
/**
 * 自定义分区器
 */
public class CustomPartitioner implements Partitioner {

    private int passLine;

    @Override
    public void configure(Map<String, ?> configs) {
        /*从生产者配置中获取分数线*/
        passLine = (Integer) configs.get("pass.line");
    }

    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, 
                         byte[] valueBytes, Cluster cluster) {
        /*key 值为分数，当分数大于分数线时候，分配到 1 分区，否则分配到 0 分区*/
        return (Integer) key >= passLine ? 1 : 0;
    }

    @Override
    public void close() {
        System.out.println("分区器关闭");
    }
}
```

&emsp;需要在创建生产者时指定分区器，和分区器所需要的配置参数：

```java
public class ProducerWithPartitioner {

    public static void main(String[] args) {

        String topicName = "Kafka-Partitioner-Test";

        Properties props = new Properties();
        props.put("bootstrap.servers", "hadoop001:9092");
        props.put("key.serializer", "org.apache.kafka.common.serialization.IntegerSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        /*传递自定义分区器*/
        props.put("partitioner.class", "com.heibaiying.producers.partitioners.CustomPartitioner");
        /*传递分区器所需的参数*/
        props.put("pass.line", 6);

        Producer<Integer, String> producer = new KafkaProducer<>(props);

        for (int i = 0; i <= 10; i++) {
            String score = "score:" + i;
            ProducerRecord<Integer, String> record = new ProducerRecord<>(topicName, i, score);
            /*异步发送消息*/
            producer.send(record, (metadata, exception) ->
                    System.out.printf("%s, partition=%d, \n", score, metadata.partition()));
        }

        producer.close();
    }
}
```

#### > 测试

&emsp;需要创建一个至少有两个分区的主题：

```shell
 bin/kafka-topics.sh --create \
                    --bootstrap-server hadoop001:9092 \
                     --replication-factor 1 --partitions 2 \
                     --topic Kafka-Partitioner-Test
```

&emsp;此时输入如下，可以看到分数大于等于 6 分的都被分到 1 分区，而小于 6 分的都被分到了 0 分区。

```shell
score:6, partition=1, 
score:7, partition=1, 
score:8, partition=1, 
score:9, partition=1, 
score:10, partition=1, 
score:0, partition=0, 
score:1, partition=0, 
score:2, partition=0, 
score:3, partition=0, 
score:4, partition=0, 
score:5, partition=0, 
分区器关闭
```



### （5）生产者其他属性

&emsp;上面生产者的创建都仅指定了服务地址，键序列化器、值序列化器，实际上 Kafka 的生产者还有很多可配置属性，如下：

#### > acks

&emsp;acks 参数指定了必须要有多少个分区副本收到消息，生产者才会认为消息写入是成功的：

- **acks=0** ： 消息发送出去就认为已经成功了，不会等待任何来自服务器的响应；
- **acks=1** ： 只要集群的首领节点收到消息，生产者就会收到一个来自服务器成功响应；
- **acks=all** ：只有当所有参与复制的节点全部收到消息时，生产者才会收到一个来自服务器的成功响应。

#### > buffer.memory

&emsp;设置生产者内存缓冲区的大小。

#### > compression.type

&emsp;默认情况下，发送的消息不会被压缩。如果想要进行压缩，可以配置此参数，可选值有 snappy，gzip，lz4。

#### > retries

&emsp;发生错误后，消息重发的次数。如果达到设定值，生产者就会放弃重试并返回错误。

#### > batch.size

&emsp;当有多个消息需要被发送到同一个分区时，生产者会把它们放在同一个批次里。该参数指定了一个批次可以使用的内存大小，按照字节数计算。

#### > linger.ms

&emsp;该参数制定了生产者在发送批次之前等待更多消息加入批次的时间。

#### > clent.id

&emsp;客户端 id,服务器用来识别消息的来源。

#### > max.in.flight.requests.per.connection

&emsp;指定了生产者在收到服务器响应之前可以发送多少个消息。它的值越高，就会占用越多的内存，不过也会提升吞吐量，把它设置为 1 可以保证消息是按照发送的顺序写入服务器，即使发生了重试。

#### > timeout.ms, request.timeout.ms & metadata.fetch.timeout.ms

- timeout.ms 指定了 borker 等待同步副本返回消息的确认时间；
- request.timeout.ms 指定了生产者在发送数据时等待服务器返回响应的时间；
- metadata.fetch.timeout.ms 指定了生产者在获取元数据（比如分区首领是谁）时等待服务器返回响应的时间。

#### > max.block.ms

&emsp;指定了在调用 `send()` 方法或使用 `partitionsFor()` 方法获取元数据时生产者的阻塞时间。当生产者的发送缓冲区已满，或者没有可用的元数据时，这些方法会阻塞。在阻塞时间达到 max.block.ms 时，生产者会抛出超时异常。

#### > max.request.size

&emsp;该参数用于控制生产者发送的请求大小。它可以指发送的单个消息的最大值，也可以指单个请求里所有消息总的大小。例如，假设这个值为 1000K ，那么可以发送的单个最大消息为 1000K ，或者生产者可以在单个请求里发送一个批次，该批次包含了 1000 个消息，每个消息大小为 1K。 

#### > receive.buffer.bytes & send.buffer.byte

&emsp;这两个参数分别指定 TCP socket 接收和发送数据包缓冲区的大小，-1 代表使用操作系统的默认值。



## 三、Kafka消费者详解

### （1）消费者和消费者群组

&emsp;在 Kafka 中，消费者通常是消费者群组的一部分，多个消费者群组共同读取同一个主题时，彼此之间互不影响。Kafka 之所以要引入消费者群组这个概念是因为 Kafka 消费者经常会做一些高延迟的操作，比如把数据写到数据库或 HDFS ，或者进行耗时的计算，在这些情况下，单个消费者无法跟上数据生成的速度。此时可以增加更多的消费者，让它们分担负载，分别处理部分分区的消息，这就是 Kafka 实现横向伸缩的主要手段。 

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/10-Kafka/images/kafka-consumer01.png) 

&emsp;需要注意的是：同一个分区只能被同一个消费者群组里面的一个消费者读取，不可能存在同一个分区被同一个消费者群里多个消费者共同读取的情况，如图：

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/10-Kafka/images/kafka-consumer02.png) 

&emsp;可以看到即便消费者 Consumer5 空闲了，但是也不会去读取任何一个分区的数据，这同时也提醒我们在使用时应该合理设置消费者的数量，以免造成闲置和额外开销。

### （2）分区再均衡

&emsp;因为群组里的消费者共同读取主题的分区，所以当一个消费者被关闭或发生崩溃时，它就离开了群组，原本由它读取的分区将由群组里的其他消费者来读取。同时在主题发生变化时 ， 比如添加了新的分区，也会发生分区与消费者的重新分配，分区的所有权从一个消费者转移到另一个消费者，这样的行为被称为再均衡。正是因为再均衡，所以消费费者群组才能保证高可用性和伸缩性。

&emsp;消费者通过向群组协调器所在的 broker 发送心跳来维持它们和群组的从属关系以及它们对分区的所有权。只要消费者以正常的时间间隔发送心跳，就被认为是活跃的，说明它还在读取分区里的消息。消费者会在轮询消息或提交偏移量时发送心跳。如果消费者停止发送心跳的时间足够长，会话就会过期，群组协调器认为它已经死亡，就会触发再均衡。

### （3）创建Kafka消费者

&emsp;在创建消费者的时候以下以下三个选项是必选的：

- **bootstrap.servers** ：指定 broker 的地址清单，清单里不需要包含所有的 broker 地址，生产者会从给定的 broker 里查找 broker 的信息。不过建议至少要提供两个 broker 的信息作为容错；
- **key.deserializer** ：指定键的反序列化器；
- **value.deserializer** ：指定值的反序列化器。

&emsp;除此之外你还需要指明你需要想订阅的主题，可以使用如下两个 API :

- **consumer.subscribe(Collection\<String> topics)**  ：指明需要订阅的主题的集合；
- **consumer.subscribe(Pattern pattern)**  ：使用正则来匹配需要订阅的集合。

&emsp;最后只需要通过轮询 API(`poll`) 向服务器定时请求数据。一旦消费者订阅了主题，轮询就会处理所有的细节，包括群组协调、分区再均衡、发送心跳和获取数据，这使得开发者只需要关注从分区返回的数据，然后进行业务处理。 示例如下：

```scala
String topic = "Hello-Kafka";
String group = "group1";
Properties props = new Properties();
props.put("bootstrap.servers", "hadoop001:9092");
/*指定分组 ID*/
props.put("group.id", group);
props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

/*订阅主题 (s)*/
consumer.subscribe(Collections.singletonList(topic));

try {
    while (true) {
        /*轮询获取数据*/
        ConsumerRecords<String, String> records = consumer.poll(Duration.of(100, ChronoUnit.MILLIS));
        for (ConsumerRecord<String, String> record : records) {
            System.out.printf("topic = %s,partition = %d, key = %s, value = %s, offset = %d,\n",
           record.topic(), record.partition(), record.key(), record.value(), record.offset());
        }
    }
} finally {
    consumer.close();
}
```

### （4）自动提交偏移量

#### > 偏移量的重要性

&emsp;Kafka 的每一条消息都有一个偏移量属性，记录了其在分区中的位置，偏移量是一个单调递增的整数。消费者通过往一个叫作 `＿consumer_offset` 的特殊主题发送消息，消息里包含每个分区的偏移量。 如果消费者一直处于运行状态，那么偏移量就没有什么用处。不过，如果有消费者退出或者新分区加入，此时就会触发再均衡。完成再均衡之后，每个消费者可能分配到新的分区，而不是之前处理的那个。为了能够继续之前的工作，消费者需要读取每个分区最后一次提交的偏移量，然后从偏移量指定的地方继续处理。 因为这个原因，所以如果不能正确提交偏移量，就可能会导致数据丢失或者重复出现消费，比如下面情况：

- 如果提交的偏移量小于客户端处理的最后一个消息的偏移量 ，那么处于两个偏移量之间的消息就会被重复消费；
- 如果提交的偏移量大于客户端处理的最后一个消息的偏移量，那么处于两个偏移量之间的消息将会丢失。

#### > 自动提交偏移量

&emsp;Kafka 支持自动提交和手动提交偏移量两种方式。这里先介绍比较简单的自动提交：

&emsp;只需要将消费者的 `enable.auto.commit` 属性配置为 `true` 即可完成自动提交的配置。 此时每隔固定的时间，消费者就会把 `poll()` 方法接收到的最大偏移量进行提交，提交间隔由 `auto.commit.interval.ms` 属性进行配置，默认值是 5s。

&emsp;使用自动提交是存在隐患的，假设我们使用默认的 5s 提交时间间隔，在最近一次提交之后的 3s 发生了再均衡，再均衡之后，消费者从最后一次提交的偏移量位置开始读取消息。这个时候偏移量已经落后了 3s ，所以在这 3s 内到达的消息会被重复处理。可以通过修改提交时间间隔来更频繁地提交偏移量，减小可能出现重复消息的时间窗，不过这种情况是无法完全避免的。基于这个原因，Kafka 也提供了手动提交偏移量的 API，使得用户可以更为灵活的提交偏移量。



### （5）手动提交偏移量

&emsp;用户可以通过将 `enable.auto.commit` 设为 `false`，然后手动提交偏移量。基于用户需求手动提交偏移量可以分为两大类：

- 手动提交当前偏移量：即手动提交当前轮询的最大偏移量；
- 手动提交固定偏移量：即按照业务需求，提交某一个固定的偏移量。

&emsp;而按照 Kafka API，手动提交偏移量又可以分为同步提交和异步提交。

#### > 同步提交

&emsp;通过调用 `consumer.commitSync()` 来进行同步提交，不传递任何参数时提交的是当前轮询的最大偏移量。

```java
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.of(100, ChronoUnit.MILLIS));
    for (ConsumerRecord<String, String> record : records) {
        System.out.println(record);
    }
    /*同步提交*/
    consumer.commitSync();
}
```

&emsp;如果某个提交失败，同步提交还会进行重试，这可以保证数据能够最大限度提交成功，但是同时也会降低程序的吞吐量。基于这个原因，Kafka 还提供了异步提交的 API。

#### > 异步提交

&emsp;异步提交可以提高程序的吞吐量，因为此时你可以尽管请求数据，而不用等待 Broker 的响应。代码如下：

```java
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.of(100, ChronoUnit.MILLIS));
    for (ConsumerRecord<String, String> record : records) {
        System.out.println(record);
    }
    /*异步提交并定义回调*/
    consumer.commitAsync(new OffsetCommitCallback() {
        @Override
        public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets, Exception exception) {
          if (exception != null) {
             System.out.println("错误处理");
             offsets.forEach((x, y) -> System.out.printf("topic = %s,partition = %d, offset = %s \n",
                                                            x.topic(), x.partition(), y.offset()));
            }
        }
    });
}
```

&emsp;异步提交存在的问题是，在提交失败的时候不会进行自动重试，实际上也不能进行自动重试。假设程序同时提交了 200 和 300 的偏移量，此时 200 的偏移量失败的，但是紧随其后的 300 的偏移量成功了，此时如果重试就会存在 200 覆盖 300 偏移量的可能。同步提交就不存在这个问题，因为在同步提交的情况下，300 的提交请求必须等待服务器返回 200 提交请求的成功反馈后才会发出。基于这个原因，某些情况下，需要同时组合同步和异步两种提交方式。

> 注：虽然程序不能在失败时候进行自动重试，但是我们是可以手动进行重试的，你可以通过一个 Map<TopicPartition, Integer> offsets 来维护你提交的每个分区的偏移量，然后当失败时候，你可以判断失败的偏移量是否小于你维护的同主题同分区的最后提交的偏移量，如果小于则代表你已经提交了更大的偏移量请求，此时不需要重试，否则就可以进行手动重试。

#### > 同步加异步提交

&emsp;下面这种情况，在正常的轮询中使用异步提交来保证吞吐量，但是因为在最后即将要关闭消费者了，所以此时需要用同步提交来保证最大限度的提交成功。

```scala
try {
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.of(100, ChronoUnit.MILLIS));
        for (ConsumerRecord<String, String> record : records) {
            System.out.println(record);
        }
        // 异步提交
        consumer.commitAsync();
    }
} catch (Exception e) {
    e.printStackTrace();
} finally {
    try {
        // 因为即将要关闭消费者，所以要用同步提交保证提交成功
        consumer.commitSync();
    } finally {
        consumer.close();
    }
}
```

#### > 提交特定偏移量

&emsp;在上面同步和异步提交的 API 中，实际上我们都没有对 commit 方法传递参数，此时默认提交的是当前轮询的最大偏移量，如果你需要提交特定的偏移量，可以调用它们的重载方法。

```java
/*同步提交特定偏移量*/
commitSync(Map<TopicPartition, OffsetAndMetadata> offsets) 
/*异步提交特定偏移量*/    
commitAsync(Map<TopicPartition, OffsetAndMetadata> offsets, OffsetCommitCallback callback)
```

&emsp;需要注意的是，因为你可以订阅多个主题，所以 `offsets` 中必须要包含所有主题的每个分区的偏移量，示例代码如下：

```java
try {
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.of(100, ChronoUnit.MILLIS));
        for (ConsumerRecord<String, String> record : records) {
            System.out.println(record);
            /*记录每个主题的每个分区的偏移量*/
            TopicPartition topicPartition = new TopicPartition(record.topic(), record.partition());
            OffsetAndMetadata offsetAndMetadata = new OffsetAndMetadata(record.offset()+1, "no metaData");
            /*TopicPartition 重写过 hashCode 和 equals 方法，所以能够保证同一主题和分区的实例不会被重复添加*/
            offsets.put(topicPartition, offsetAndMetadata);
        }
        /*提交特定偏移量*/
        consumer.commitAsync(offsets, null);
    }
} finally {
    consumer.close();
}
```



### （6）监听分区再均衡

&emsp;因为分区再均衡会导致分区与消费者的重新划分，有时候你可能希望在再均衡前执行一些操作：比如提交已经处理但是尚未提交的偏移量，关闭数据库连接等。此时可以在订阅主题时候，调用 `subscribe` 的重载方法传入自定义的分区再均衡监听器。

```java
 /*订阅指定集合内的所有主题*/
subscribe(Collection<String> topics, ConsumerRebalanceListener listener)
 /*使用正则匹配需要订阅的主题*/    
subscribe(Pattern pattern, ConsumerRebalanceListener listener)    
```

&emsp;代码示例如下：

```java
Map<TopicPartition, OffsetAndMetadata> offsets = new HashMap<>();

consumer.subscribe(Collections.singletonList(topic), new ConsumerRebalanceListener() {
    /*该方法会在消费者停止读取消息之后，再均衡开始之前就调用*/
    @Override
    public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
        System.out.println("再均衡即将触发");
        // 提交已经处理的偏移量
        consumer.commitSync(offsets);
    }

    /*该方法会在重新分配分区之后，消费者开始读取消息之前被调用*/
    @Override
    public void onPartitionsAssigned(Collection<TopicPartition> partitions) {

    }
});

try {
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.of(100, ChronoUnit.MILLIS));
        for (ConsumerRecord<String, String> record : records) {
            System.out.println(record);
            TopicPartition topicPartition = new TopicPartition(record.topic(), record.partition());
            OffsetAndMetadata offsetAndMetadata = new OffsetAndMetadata(record.offset() + 1, "no metaData");
            /*TopicPartition 重写过 hashCode 和 equals 方法，所以能够保证同一主题和分区的实例不会被重复添加*/
            offsets.put(topicPartition, offsetAndMetadata);
        }
        consumer.commitAsync(offsets, null);
    }
} finally {
    consumer.close();
}
```



### （7）退出轮询

&emsp;Kafka 提供了 `consumer.wakeup()` 方法用于退出轮询，它通过抛出 `WakeupException` 异常来跳出循环。需要注意的是，在退出线程时最好显示的调用 `consumer.close()` , 此时消费者会提交任何还没有提交的东西，并向群组协调器发送消息，告知自己要离开群组，接下来就会触发再均衡 ，而不需要等待会话超时。 

下面的示例代码为监听控制台输出，当输入 `exit` 时结束轮询，关闭消费者并退出程序：

```java
/*调用 wakeup 优雅的退出*/
final Thread mainThread = Thread.currentThread();
new Thread(() -> {
    Scanner sc = new Scanner(System.in);
    while (sc.hasNext()) {
        if ("exit".equals(sc.next())) {
            consumer.wakeup();
            try {
                /*等待主线程完成提交偏移量、关闭消费者等操作*/
                mainThread.join();
                break;
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}).start();

try {
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.of(100, ChronoUnit.MILLIS));
        for (ConsumerRecord<String, String> rd : records) {
            System.out.printf("topic = %s,partition = %d, key = %s, value = %s, offset = %d,\n",
                              rd.topic(), rd.partition(), rd.key(), rd.value(), rd.offset());
        }
    }
} catch (WakeupException e) {
    //对于 wakeup() 调用引起的 WakeupException 异常可以不必处理
} finally {
    consumer.close();
    System.out.println("consumer 关闭");
}
```



### （8）独立的消费者

&emsp;因为 Kafka 的设计目标是高吞吐和低延迟，所以在 Kafka 中，消费者通常都是从属于某个群组的，这是因为单个消费者的处理能力是有限的。但是某些时候你的需求可能很简单，比如可能只需要一个消费者从一个主题的所有分区或者某个特定的分区读取数据，这个时候就不需要消费者群组和再均衡了， 只需要把主题或者分区分配给消费者，然后开始读取消息井提交偏移量即可。

&emsp;在这种情况下，就不需要订阅主题， 取而代之的是消费者为自己分配分区。 一个消费者可以订阅主题（井加入消费者群组），或者为自己分配分区，但不能同时做这两件事情。 分配分区的示例代码如下：

```java
List<TopicPartition> partitions = new ArrayList<>();
List<PartitionInfo> partitionInfos = consumer.partitionsFor(topic);

/*可以指定读取哪些分区 如这里假设只读取主题的 0 分区*/
for (PartitionInfo partition : partitionInfos) {
    if (partition.partition()==0){
        partitions.add(new TopicPartition(partition.topic(), partition.partition()));
    }
}

// 为消费者指定分区
consumer.assign(partitions);


while (true) {
    ConsumerRecords<Integer, String> records = consumer.poll(Duration.of(100, ChronoUnit.MILLIS));
    for (ConsumerRecord<Integer, String> record : records) {
        System.out.printf("partition = %s, key = %d, value = %s\n",
                          record.partition(), record.key(), record.value());
    }
    consumer.commitSync();
}
```



### （9）Kafka消费者可选属性

#### > fetch.min.byte

&emsp;消费者从服务器获取记录的最小字节数。如果可用的数据量小于设置值，broker 会等待有足够的可用数据时才会把它返回给消费者。

#### > fetch.max.wait.ms

&emsp;broker 返回给消费者数据的等待时间，默认是 500ms。

#### > max.partition.fetch.bytes

&emsp;该属性指定了服务器从每个分区返回给消费者的最大字节数，默认为 1MB。

#### > session.timeout.ms

&emsp;消费者在被认为死亡之前可以与服务器断开连接的时间，默认是 3s。

#### > auto.offset.reset

&emsp;该属性指定了消费者在读取一个没有偏移量的分区或者偏移量无效的情况下该作何处理：

- latest (默认值) ：在偏移量无效的情况下，消费者将从最新的记录开始读取数据（在消费者启动之后生成的最新记录）;
- earliest ：在偏移量无效的情况下，消费者将从起始位置读取分区的记录。

#### > enable.auto.commit

&emsp;是否自动提交偏移量，默认值是 true。为了避免出现重复消费和数据丢失，可以把它设置为 false。

#### > client.id

&emsp;客户端 id，服务器用来识别消息的来源。

#### > max.poll.records

&emsp;单次调用 `poll()` 方法能够返回的记录数量。

#### > receive.buffer.bytes & send.buffer.byte

&emsp;这两个参数分别指定 TCP socket 接收和发送数据包缓冲区的大小，-1 代表使用操作系统的默认值。



## 四、深入理解Kafka副本机制

### （1）Kafka集群

&emsp;Kafka 使用 Zookeeper 来维护集群成员 (brokers) 的信息。每个 broker 都有一个唯一标识 `broker.id`，用于标识自己在集群中的身份，可以在配置文件 `server.properties` 中进行配置，或者由程序自动生成。下面是 Kafka brokers 集群自动创建的过程：

- 每一个 broker 启动的时候，它会在 Zookeeper 的 `/brokers/ids` 路径下创建一个 ` 临时节点 `，并将自己的 `broker.id` 写入，从而将自身注册到集群；
- 当有多个 broker 时，所有 broker 会竞争性地在 Zookeeper 上创建 `/controller` 节点，由于 Zookeeper 上的节点不会重复，所以必然只会有一个 broker 创建成功，此时该 broker 称为 controller broker。它除了具备其他 broker 的功能外，**还负责管理主题分区及其副本的状态**。
- 当 broker 出现宕机或者主动退出从而导致其持有的 Zookeeper 会话超时时，会触发注册在 Zookeeper 上的 watcher 事件，此时 Kafka 会进行相应的容错处理；如果宕机的是 controller broker 时，还会触发新的 controller 选举。

### （2）副本机制

&emsp;为了保证高可用，kafka 的分区是多副本的，如果一个副本丢失了，那么还可以从其他副本中获取分区数据。但是这要求对应副本的数据必须是完整的，这是 Kafka 数据一致性的基础，所以才需要使用 `controller broker` 来进行专门的管理。下面将详解介绍 Kafka 的副本机制。

#### > 分区和副本

&emsp;Kafka 的主题被分为多个分区 ，分区是 Kafka 最基本的存储单位。每个分区可以有多个副本 (可以在创建主题时使用 ` replication-factor` 参数进行指定)。其中一个副本是首领副本 (Leader replica)，所有的事件都直接发送给首领副本；其他副本是跟随者副本 (Follower replica)，需要通过复制来保持与首领副本数据一致，当首领副本不可用时，其中一个跟随者副本将成为新首领。 

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/10-Kafka/images/kafka-cluster.png)

#### > ISR机制

&emsp;每个分区都有一个 ISR(in-sync Replica) 列表，用于维护所有同步的、可用的副本。首领副本必然是同步副本，而对于跟随者副本来说，它需要满足以下条件才能被认为是同步副本：

- 与 Zookeeper 之间有一个活跃的会话，即必须定时向 Zookeeper 发送心跳；
- 在规定的时间内从首领副本那里低延迟地获取过消息。

&emsp;如果副本不满足上面条件的话，就会被从 ISR 列表中移除，直到满足条件才会被再次加入。

&emsp;这里给出一个主题创建的示例：使用 `--replication-factor` 指定副本系数为 3，创建成功后使用 `--describe ` 命令可以看到分区 0 的有 0,1,2 三个副本，且三个副本都在 ISR 列表中，其中 1 为首领副本。

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/10-Kafka/images/kafka-分区副本.png) 

#### > 不完全的首领选举

&emsp;对于副本机制，在 broker 级别有一个可选的配置参数 `unclean.leader.election.enable`，默认值为 fasle，代表禁止不完全的首领选举。这是针对当首领副本挂掉且 ISR 中没有其他可用副本时，是否允许某个不完全同步的副本成为首领副本，这可能会导致数据丢失或者数据不一致，在某些对数据一致性要求较高的场景 (如金融领域)，这可能无法容忍的，所以其默认值为 false，如果你能够允许部分数据不一致的话，可以配置为 true。

#### > 最少同步副本

&emsp;ISR 机制的另外一个相关参数是 `min.insync.replicas` , 可以在 broker 或者主题级别进行配置，代表 ISR 列表中至少要有几个可用副本。这里假设设置为 2，那么当可用副本数量小于该值时，就认为整个分区处于不可用状态。此时客户端再向分区写入数据时候就会抛出异常 `org.apache.kafka.common.errors.NotEnoughReplicasExceptoin: Messages are rejected since there are fewer in-sync replicas than required。`

#### > 发送确认

&emsp;Kafka 在生产者上有一个可选的参数 ack，该参数指定了必须要有多少个分区副本收到消息，生产者才会认为消息写入成功：

- **acks=0** ：消息发送出去就认为已经成功了，不会等待任何来自服务器的响应；
- **acks=1** ： 只要集群的首领节点收到消息，生产者就会收到一个来自服务器成功响应；
- **acks=all** ：只有当所有参与复制的节点全部收到消息时，生产者才会收到一个来自服务器的成功响应。

### （3）数据请求

#### > 元数据请求机制

&emsp;在所有副本中，只有领导副本才能进行消息的读写处理。由于不同分区的领导副本可能在不同的 broker 上，如果某个 broker 收到了一个分区请求，但是该分区的领导副本并不在该 broker 上，那么它就会向客户端返回一个 `Not a Leader for Partition` 的错误响应。 为了解决这个问题，Kafka 提供了元数据请求机制。

&emsp;首先集群中的每个 broker 都会缓存所有主题的分区副本信息，客户端会定期发送发送元数据请求，然后将获取的元数据进行缓存。定时刷新元数据的时间间隔可以通过为客户端配置 `metadata.max.age.ms` 来进行指定。有了元数据信息后，客户端就知道了领导副本所在的 broker，之后直接将读写请求发送给对应的 broker 即可。

&emsp;如果在定时请求的时间间隔内发生的分区副本的选举，则意味着原来缓存的信息可能已经过时了，此时还有可能会收到 `Not a Leader for Partition` 的错误响应，这种情况下客户端会再次求发出元数据请求，然后刷新本地缓存，之后再去正确的 broker 上执行对应的操作，过程如下图：

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/10-Kafka/images/kafka-元数据请求.png) 

#### > 数据可见性

&emsp;需要注意的是，并不是所有保存在分区首领上的数据都可以被客户端读取到，为了保证数据一致性，只有被所有同步副本 (ISR 中所有副本) 都保存了的数据才能被客户端读取到。

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/10-Kafka/images/kafka-数据可见性.png) 

#### > 零拷贝

&emsp;Kafka 所有数据的写入和读取都是通过零拷贝来实现的。传统拷贝与零拷贝的区别如下：

- 传统模式下的四次拷贝与四次上下文切换

&emsp;以将磁盘文件通过网络发送为例。传统模式下，一般使用如下伪代码所示的方法先将文件数据读入内存，然后通过 Socket 将内存中的数据发送出去。

```java
buffer = File.read
Socket.send(buffer)
```

&emsp;这一过程实际上发生了四次数据拷贝。首先通过系统调用将文件数据读入到内核态 Buffer（DMA 拷贝），然后应用程序将内存态 Buffer 数据读入到用户态 Buffer（CPU 拷贝），接着用户程序通过 Socket 发送数据时将用户态 Buffer 数据拷贝到内核态 Buffer（CPU 拷贝），最后通过 DMA 拷贝将数据拷贝到 NIC Buffer。同时，还伴随着四次上下文切换，如下图所示：

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/10-Kafka/images/kafka-BIO.png) 

- sendfile和transferTo实现零拷贝

&emsp;Linux 2.4+ 内核通过 `sendfile` 系统调用，提供了零拷贝。数据通过 DMA 拷贝到内核态 Buffer 后，直接通过 DMA 拷贝到 NIC Buffer，无需 CPU 拷贝。这也是零拷贝这一说法的来源。除了减少数据拷贝外，因为整个读文件到网络发送由一个 `sendfile` 调用完成，整个过程只有两次上下文切换，因此大大提高了性能。零拷贝过程如下图所示：

![img](./images/kafka-零拷贝.png) 

&emsp;从具体实现来看，Kafka 的数据传输通过 TransportLayer 来完成，其子类 `PlaintextTransportLayer` 的 `transferFrom` 方法通过调用 Java NIO 中 FileChannel 的 `transferTo` 方法实现零拷贝，如下所示：

```java
@Override
public long transferFrom(FileChannel fileChannel, long position, long count) throws IOException {
    return fileChannel.transferTo(position, count, socketChannel);
}
```

&emsp;**注：** `transferTo` 和 `transferFrom` 并不保证一定能使用零拷贝。实际上是否能使用零拷贝与操作系统相关，如果操作系统提供 `sendfile` 这样的零拷贝系统调用，则这两个方法会通过这样的系统调用充分利用零拷贝的优势，否则并不能通过这两个方法本身实现零拷贝。

### （4）物理存储

#### > 分区分配

&emsp;在创建主题时，Kafka 会首先决定如何在 broker 间分配分区副本，它遵循以下原则：

- 在所有 broker 上均匀地分配分区副本；
- 确保分区的每个副本分布在不同的 broker 上；
- 如果使用了 `broker.rack` 参数为 broker 指定了机架信息，那么会尽可能的把每个分区的副本分配到不同机架的 broker 上，以避免一个机架不可用而导致整个分区不可用。

&emsp;基于以上原因，如果你在一个单节点上创建一个 3 副本的主题，通常会抛出下面的异常：

```properties
Error while executing topic command : org.apache.kafka.common.errors.InvalidReplicationFactor   
Exception: Replication factor: 3 larger than available brokers: 1.
```

#### > 分区数据保留规则

&emsp;保留数据是 Kafka 的一个基本特性， 但是 Kafka 不会一直保留数据，也不会等到所有消费者都读取了消息之后才删除消息。相反， Kafka 为每个主题配置了数据保留期限，规定数据被删除之前可以保留多长时间，或者清理数据之前可以保留的数据量大小。分别对应以下四个参数： 

- `log.retention.bytes` ：删除数据前允许的最大数据量；默认值-1，代表没有限制；
- `log.retention.ms`：保存数据文件的毫秒数，如果未设置，则使用 `log.retention.minutes` 中的值，默认为 null；
- `log.retention.minutes`：保留数据文件的分钟数，如果未设置，则使用 `log.retention.hours` 中的值，默认为 null；
- `log.retention.hours`：保留数据文件的小时数，默认值为 168，也就是一周。

&emsp;因为在一个大文件里查找和删除消息是很费时的，也很容易出错，所以 Kafka 把分区分成若干个片段，当前正在写入数据的片段叫作活跃片段。活动片段永远不会被删除。如果按照默认值保留数据一周，而且每天使用一个新片段，那么你就会看到，在每天使用一个新片段的同时会删除一个最老的片段，所以大部分时间该分区会有 7 个片段存在。 

#### > 文件格式

&emsp;通常保存在磁盘上的数据格式与生产者发送过来消息格式是一样的。 如果生产者发送的是压缩过的消息，那么同一个批次的消息会被压缩在一起，被当作“包装消息”进行发送 (格式如下所示) ，然后保存到磁盘上。之后消费者读取后再自己解压这个包装消息，获取每条消息的具体信息。

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/10-Kafka/images/kafka-compress-message.png) 



## 五、基于Zookeeper搭建Kafka高可用集群

> Zookeeper 在 在 Kafka  中的作用:
>
> &emsp;Kafka 集群中有一个 broker 会被选举为 Controller，负责管理集群 broker 的上下线，所有 topic 的分区副本分配和 leader 选举等工作。
>
> &emsp;Controller 的管理工作都是依赖于 Zookeeper 的。

### （1）Zookeeper集群搭建

&emsp;为保证集群高可用，Zookeeper 集群的节点数最好是奇数，最少有三个节点，所以这里搭建一个三个节点的集群。

#### > 下载 & 解压

&emsp;下载对应版本 Zookeeper，这里我下载的版本 `3.4.14`。官方下载地址：https://archive.apache.org/dist/zookeeper/

```shell
# 下载
wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz
# 解压
tar -zxvf zookeeper-3.4.14.tar.gz
```

#### > 修改配置

&emsp;拷贝三份 zookeeper 安装包。分别进入安装目录的 `conf` 目录，拷贝配置样本 `zoo_sample.cfg ` 为 `zoo.cfg` 并进行修改，修改后三份配置文件内容分别如下：

zookeeper01 配置：

```shell
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/usr/local/zookeeper-cluster/data/01
dataLogDir=/usr/local/zookeeper-cluster/log/01
clientPort=2181

# server.1 这个1是服务器的标识，可以是任意有效数字，标识这是第几个服务器节点，这个标识要写到dataDir目录下面myid文件里
# 指名集群间通讯端口和选举端口
server.1=127.0.0.1:2287:3387
server.2=127.0.0.1:2288:3388
server.3=127.0.0.1:2289:3389
```

> 如果是多台服务器，则集群中每个节点通讯端口和选举端口可相同，IP 地址修改为每个节点所在主机 IP 即可。

&emsp;zookeeper02 配置，与 zookeeper01 相比，只有 `dataLogDir`、`dataLogDir` 和 `clientPort` 不同：

```shell
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/usr/local/zookeeper-cluster/data/02
dataLogDir=/usr/local/zookeeper-cluster/log/02
clientPort=2182

server.1=127.0.0.1:2287:3387
server.2=127.0.0.1:2288:3388
server.3=127.0.0.1:2289:3389
```

&emsp;zookeeper03 配置，与 zookeeper01，02 相比，也只有 `dataLogDir`、`dataLogDir` 和 `clientPort` 不同：

```shell
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/usr/local/zookeeper-cluster/data/03
dataLogDir=/usr/local/zookeeper-cluster/log/03
clientPort=2183

server.1=127.0.0.1:2287:3387
server.2=127.0.0.1:2288:3388
server.3=127.0.0.1:2289:3389
```

> 配置参数说明：
>
> - **tickTime**：用于计算的基础时间单元。比如 session 超时：N*tickTime；
> - **initLimit**：用于集群，允许从节点连接并同步到 master 节点的初始化连接时间，以 tickTime 的倍数来表示；
> - **syncLimit**：用于集群， master 主节点与从节点之间发送消息，请求和应答时间长度（心跳机制）；
> - **dataDir**：数据存储位置；
> - **dataLogDir**：日志目录；
> - **clientPort**：用于客户端连接的端口，默认 2181



#### > 标识节点

&emsp;分别在三个节点的数据存储目录下新建 `myid` 文件,并写入对应的节点标识。Zookeeper 集群通过 `myid` 文件识别集群节点，并通过上文配置的节点通信端口和选举端口来进行节点通信，选举出 leader 节点。

&emsp;创建存储目录：

```shell
# dataDir
mkdir -vp  /usr/local/zookeeper-cluster/data/01
# dataDir
mkdir -vp  /usr/local/zookeeper-cluster/data/02
# dataDir
mkdir -vp  /usr/local/zookeeper-cluster/data/03
```

&emsp;创建并写入节点标识到 `myid` 文件：

```shell
#server1
echo "1" > /usr/local/zookeeper-cluster/data/01/myid
#server2
echo "2" > /usr/local/zookeeper-cluster/data/02/myid
#server3
echo "3" > /usr/local/zookeeper-cluster/data/03/myid
```

#### > 启动集群

&emsp;分别启动三个节点：

```shell
# 启动节点1
/usr/app/zookeeper-cluster/zookeeper01/bin/zkServer.sh start
# 启动节点2
/usr/app/zookeeper-cluster/zookeeper02/bin/zkServer.sh start
# 启动节点3
/usr/app/zookeeper-cluster/zookeeper03/bin/zkServer.sh start
```

#### > 集群验证

&emsp;使用 jps 查看进程，并且使用 `zkServer.sh status` 查看集群各个节点状态。如图三个节点进程均启动成功，并且两个节点为 follower 节点，一个节点为 leader 节点。

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/10-Kafka/images/zookeeper-cluster.png) 



### （2）Kafka集群搭建

#### > 下载解压

&emsp;Kafka 安装包官方下载地址：http://kafka.apache.org/downloads ，本用例下载的版本为 `2.2.0`，下载命令：

```shell
# 下载
wget https://www-eu.apache.org/dist/kafka/2.2.0/kafka_2.12-2.2.0.tgz
# 解压
tar -xzf kafka_2.12-2.2.0.tgz
```

> 这里 j 解释一下 kafka 安装包的命名规则：以 `kafka_2.12-2.2.0.tgz` 为例，前面的 2.12 代表 Scala 的版本号（Kafka 采用 Scala 语言进行开发），后面的 2.2.0 则代表 Kafka 的版本号。

#### > 拷贝配置文件

&emsp;进入解压目录的 ` config` 目录下 ，拷贝三份配置文件：

```shell
# cp server.properties server-1.properties
# cp server.properties server-2.properties
# cp server.properties server-3.properties
```

#### > 修改配置

&emsp;分别修改三份配置文件中的部分配置，如下：

&emsp;server-1.properties：

```properties
# The id of the broker. 集群中每个节点的唯一标识
broker.id=0
# 监听地址
listeners=PLAINTEXT://hadoop001:9092
# 数据的存储位置
log.dirs=/usr/local/kafka-logs/00
# Zookeeper连接地址
zookeeper.connect=hadoop001:2181,hadoop001:2182,hadoop001:2183
```

&emsp;server-2.properties：

```properties
broker.id=1
listeners=PLAINTEXT://hadoop001:9093
log.dirs=/usr/local/kafka-logs/01
zookeeper.connect=hadoop001:2181,hadoop001:2182,hadoop001:2183
```

&emsp;server-3.properties：

```properties
broker.id=2
listeners=PLAINTEXT://hadoop001:9094
log.dirs=/usr/local/kafka-logs/02
zookeeper.connect=hadoop001:2181,hadoop001:2182,hadoop001:2183
```

&emsp;这里需要说明的是 `log.dirs` 指的是数据日志的存储位置，确切的说，就是分区数据的存储位置，而不是程序运行日志的位置。程序运行日志的位置是通过同一目录下的 `log4j.properties` 进行配置的。

#### > 启动集群

&emsp;分别指定不同配置文件，启动三个 Kafka 节点。启动后可以使用 jps 查看进程，此时应该有三个 zookeeper 进程和三个 kafka 进程。

```shell
bin/kafka-server-start.sh config/server-1.properties
bin/kafka-server-start.sh config/server-2.properties
bin/kafka-server-start.sh config/server-3.properties
```

#### > 创建测试主题

&emsp;创建测试主题：

```shell
bin/kafka-topics.sh --create --bootstrap-server hadoop001:9092 \
					--replication-factor 3 \
					--partitions 1 --topic my-replicated-topic
```

&emsp;创建后可以使用以下命令查看创建的主题信息：

```shell
bin/kafka-topics.sh --describe --bootstrap-server hadoop001:9092 --topic my-replicated-topic
```

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/01-大数据相关技术栈/10-Kafka/images/kafka-cluster-shell.png) 



&emsp;可以看到分区 0 的有 0,1,2 三个副本，且三个副本都是可用副本，都在 ISR(in-sync Replica 同步副本) 列表中，其中 1 为首领副本，此时代表集群已经搭建成功。

### （3）Kafka 命令行

- **查看当前服务器中的所有 topic** 

  ```shell
  [atguigu@hadoop102  kafka]$  bin/kafka-topics.sh  --zookeeper
  hadoop102:2181 --list
  ```

- **创建 topic**

  ```shell
  [root@hadoop102  kafka]$  bin/kafka-topics.sh  --zookeeper
  hadoop102:2181 --create --replication-factor 3 --partitions 1 --
  topic first
  ```

  选项说明：

  > --topic 定义 topic 名
  >
  > --replication-factor 定义副本数
  >
  > --partitions 定义分区数

- **删除 topic**

  ```shell
  [root@hadoop102  kafka]$  bin/kafka-topics.sh  --zookeeper
  hadoop102:2181 --delete --topic first
  ```

  需要 server.properties 中设置 delete.topic.enable=true 否则只是标记删除。

- **发送消息**

  ```shell
  [root@hadoop102 kafka]$ bin/kafka-console-producer.sh --broker-
  list hadoop102:9092 --topic first
  >hello world
  ```

- **消费消息**

  ```shell
  [root@hadoop102 kafka]$ bin/kafka-console-consumer.sh \
  --zookeeper hadoop102:2181 --topic first

  [root@hadoop102 kafka]$ bin/kafka-console-consumer.sh \
  --bootstrap-server hadoop102:9092 --topic first

  [root@hadoop102 kafka]$ bin/kafka-console-consumer.sh \
  --bootstrap-server hadoop102:9092 --from-beginning --topic first
  ```

  --from-beginning：会把主题中以往所有的数据都读取出来。

- **查看某个 Topic 的详情**

  ```shell
  [root@hadoop102  kafka]$  bin/kafka-topics.sh  --zookeeper
  hadoop102:2181 --describe --topic first
  ```

- **修改分区数**

  ```shell
  [root@hadoop102  kafka]$  bin/kafka-topics.sh  --zookeeper
  hadoop102:2181 --alter --topic first --partitions 6
  ```

  ​

  ## 六、Kafka 面试题

  1.Kafka 中的 ISR(InSyncRepli)、OSR(OutSyncRepli)、AR(AllRepli)又代表什么？

  2.Kafka 中的 HW、LEO 等分别代表什么？

  3.Kafka 中是怎么体现消息顺序性的？

  4.Kafka 中的分区器、序列化器、拦截器是否了解？它们之间的处理顺序是什么？

  5.Kafka 生产者客户端的整体结构是什么样子的？使用了几个线程来处理？分别是什么？

  6.“消费组中的消费者个数如果超过 topic 的分区，那么就会有消费者消费不到数据”这句话是否正确？

  7.消费者提交消费位移时提交的是当前消费到的最新消息的 offset 还是 offset+1？

  8.有哪些情形会造成重复消费？

  9.那些情景会造成消息漏消费？

  10.当你使用 kafka-topics.sh 创建（删除）了一个 topic 之后，Kafka 背后会执行什么逻辑？

  &emsp;1）会在 zookeeper 中的/brokers/topics 节点下创建一个新的 topic 节点，如：/brokers/topics/first

  &emsp;2）触发 Controller 的监听程序

  &emsp;3）kafka Controller 负责 topic 的创建工作，并更新 metadata cache

  11.topic 的分区数可不可以增加？如果可以怎么增加？如果不可以，那又是为什么？

  12.topic 的分区数可不可以减少？如果可以怎么减少？如果不可以，那又是为什么？

  13.Kafka 有内部的 topic 吗？如果有是什么？有什么所用？

  14.Kafka 分区分配的概念？

  15.简述 Kafka 的日志目录结构？

  16.如果我指定了一个 offset，Kafka Controller 怎么查找到对应的消息？

  17.聊一聊 Kafka Controller 的作用？

  18.Kafka 中有那些地方需要选举？这些地方的选举策略又有哪些？

  19.失效副本是指什么？有那些应对措施？

  20.Kafka 的哪些设计让它有如此高的性能？

  ​


<div align="center"> <img  src="https://gitee.com/MartinHub/MartinHub-notes/raw/master/images/weixin.png" width="200"/> </div>
# Kafka-producer 开发 
{docsify-updated}

### Java版 producer 工作流程
如下图所示：
<center><img src="pics/kafka-producer.png" alt="" width="50%"></center>
producer 首先使用一个线程将待发送的消息封装进一个 ProducerRecord 类实例，然后将其序列化之后发送给 partitioner，再由后者确定了目标分区后一同发送到位于 producer 程序内部的一块内存缓冲区中。 producer 的另一个 IO 线程则负责实时地从该缓冲区中提取出准备就绪的消息封装进一个批次，统一发送给对应的 broker 。

构造一个 Kafka producer 实例大致需要5个步骤：
1. 构造一个 `Properties` 对象，至少指定 `bootstrap.servers` 、 `key.serializer` 和 `value.serializer` 这3个属性。或者这些参数可以在构造 `KafkaProducer` 时传人构造函数。这样 `Properties` 就不用指定这些参数了。
2. 使用上一步创建的 `Properties` 实例构造 `KafkaProducer` 对象
3. 构造要发送的消息对象 `ProdecerRecord`，指定消息要被发送到的 topic、分区以及对应的 key 和 value 。注意，分区和 key 信息可以不用指定，由 Kafka 自行确定目标分区。
4. 调用 `KafkaProducer` 的 send 方法发送消息。
5. 关闭 KafkaProducer 。

Demo:
```
Properties kafkaProp = new Properties();
kafkaProp.put("bootstrap.servers","172.26.238.80:9092");
kafkaProp.put("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
kafkaProp.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
Producer<String,String> producer = new KafkaProducer<String, String>(kafkaProp);
for (int i = 0; i < 10; i++) {
   producer.send(new ProducerRecord<>("quick-start",Integer.toString(i),Integer.toString(i)));
}
```

##### producer 的主要参数
1. `acks`  
   这个参数制定了kafka服务端在给 producer 发送相应前， lead broker 端必须要确保已经写入该消息的副本的数量。当前有3个值：0、1和 all。
   + `acks=0`：表示 producer 完全不理睬 leader broker 端的处理结果。吞吐量最高，但是消息可能会写入失败。
   + `acks=all` 或者 -1: leader broker 不仅会将消息写入本地日志，同时还会等待 ISR 中的所有其他副本都成功写入它们各自的本地日志后才返回相应结果给 producer 端。这样可以保证当 ISR 中至少有一个副本是存活状态时，消息是不会丢失的。但是吞吐量也是最低的。
   + `acks=1`：是0和all的折中方案，也是默认值。leader broker 收到消息后将消息写入本地日志，然后就发送相应结果给 producer ，无需等待 ISR 中其它副本写入该消息。
    <center><img src="pics/kafka-acks.png" alt="" width="60%"></center>

2. `buffer.memory`
   该参数指定了 producer 端用于缓存消息的缓冲区的大小，单位是字节，默认值是 32MB。如前所述， producer 主线程会把要发送的消息放入缓冲区，然后由IO线程发送这些消息。
3. `compression.type`
   设置 producer 端是否压缩消息，默认不压缩。主要的压缩算法有： GZIP、Snappy、LZ4、Zstandard。
4. `retries`
   当消息发送失败时，该参数能指定重试的次数。值得注意的是重试可能造成消息重复发送和消息的乱序。
5. `batch.size`
   前面提到过， producer 会把发往同一分区的多条消息封装进一个 batch 中。当 batch 满了的时候， producer 会放 batch 中的所有消息。不过， producer 并不总是等待 batch 满了才发送消息。 batch.size 默认 16KB。
6. `linger.ms`
   该参数控制的是消息发送的延时，默认值是0，表示消息需要被立即发送，无需关心batch 是否已被填满。
7. `request.timeout.ms`
   设置的是超时时间，如果发送后，在该参数指定的时间内，producer 没有收到 broker 的响应，那么 producer 就认为该请求超时了。
8. `parttioner.class`
   设置自定义的 parttioner 分区机制。
9. `enable.idempotence`
   幂等性设置，启用以后，相同的消息（错误重试的消息）只会被集群存储一次。

#####  ProducerRecord
ProducerRecord代表了一条要发送的消息， 由5个字段构成，它们分别如下:
+ topic ：该消息所属的 topic 
+ partition ：直接指定该消息所属的分区 
+ key ：消息 key 
+ value ：消息体
+ timestamp ：消息时间戳，慎重使用这个功能，因为它有可能会令时间戳索引机制失效

#### RecordMetadata
该数据结构表示 Kafka 服务器端返回给客户端的消息的元数据信息，包含如下内容 。
+ offset ：消息在分区日志中的位移信息。
+ timestamp ：消息时间戳 。
+ topic/partition ：所属 topic 的分区 。
+ checksum ：消息 CRC32 码 。
+ serializedKeySize ：序列化后的消息 key 字节数。
+ serializedValueSize ：序列化后的消息 value 字节数 。

##### 消息分区机制
Kafka 的消息发送过程中，很重要的一步就是要确定将消息发送到指定的 topic 的哪个分区中。默认的分区策略会尽力确保具有相同 key 的所有消息都会被发送到相同的分区上；如果没有为消息指定 key ，则会选择轮询的方式来确保消息在 topic 的所有分上均匀分布。

如果用户需要自定义分区机制，就需要完成两件事：
1. 在 producer 程序中创建一个类，实现 `org.apache.kafka.clients.producer.Partitioner` 接口，主要分区逻辑在 `partition()`方法中实现。
2. 在用于构造 `KafkaProducer`的 Properties 对象中设置 `parttioner.class`参数。

### 消息序列化
序列化器负责在 producer 发送前将消息转换成字节数组；解序列化器则用于在 consumer 接收到的字节数组转换成相应的对象。Kafka 默认提供了十几种序列化器，常用的如下：
+ ByteArraySerializer: 序列化字节数组。
+ ByteBufferSerializer: 序列化 ByteBuffer。
+ ByteSerializer: 序列化 Kafka 自定义的 Bytes 类。
+ DoubleSerializer: 序列化 Double 类型。
+ IntegerSerializer：序列化 Integer 类型。
+ LongSerializer: 序列化 Long 类型。
+ StringSerializer: 序列化 String 类型。

若涉及到复杂类型的序列化，则需要用户自定义消息序列化器。编写一个自定义的 Serializer ，需要以下几步：
1. 定义数据对象格式。
2. 创建自定义的序列化类，实现 `org.apache.kafka.common.serialization.Serializer` 接口，在 `serializer` 方法中实现序列化逻辑。
3. 在用于构造 KafkaProducer 的 Properties 对象中设置 `key.serializer` 或 `value.serializer` :取决于是为消息的 key 还是 value 做自定义序列化。

### Producer 拦截器
对于 producer 而言，消息拦截器（interceptor）使得用户在消息发送之前以及收到 broker 响应执行 producer 回调逻辑之前有机会对消息做一些定制化的需求。同时允许指定多个 interceptor 按序作用于同一条消息从而形成一个拦截链。主要就是实现 `org.apache.kafka.clients.producer.ProducerInterceptor` 接口。


tips:
在 windows wsl 环境中启动 kafka 集群时，如果要在宿主机连接到该集群，必须在 kafka 配置 listener 时指定 ip 地址，在客户端连接时，也用 ip 地址连接。否则会有网络问题。
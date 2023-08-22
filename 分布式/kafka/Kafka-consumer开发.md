## Kafka-consumer 开发 
{docsify-updated}

- [Kafka-consumer 开发](#kafka-consumer-开发)
	- [消费者组与独立消费者](#消费者组与独立消费者)
	- [构建 consumer](#构建-consumer)
	- [Consumer 的主要参数](#consumer-的主要参数)
	- [位移](#位移)
	- [重平衡](#重平衡)
	- [消息交付语义](#消息交付语义)


### 消费者组与独立消费者
消费者组使用一个消费组名(group.id)来标记自己， topic 的每条消息都会被发送到每个订阅它的消费者组的一个消费者实例上：
1. 一个 group.id 唯一标识一个 consumer group 。一个 consumer group 可能有若干个 consumer 实例
2. 对于同一个 group 而言， topic 的每条消息只能被发送到 group 内的一个实例上
3. topic 消息可以被发送到多个 group 中。
4. 对某个 group 而言，订阅 topic 的每个分区只能分配给该 group 下的一个 consumer 实例（当然该分区还可以被分配给其他订阅该 topic 的消费者组）。

所以 Kafka 使用下面的两种方式来支持的基于队列和基于发布/订阅的两种消息模型：
1. 所有的 consumer 实例都划分到同一个 group -> 实现基于队列的模型，每条消息只会被一个 consumer 实例消费
2. 将所有的 consumer 实例划分到若干不同的 group -> 实现基于发布/订阅的模型。极端的情况是每个 consumer 实例都属于不同的 group，这样 Kafka 消息会被广播到所有 consume 实例上。

 Kafka 目前只提供单个分区内的消息顺序，而不会维护全局的消息顺序，因此如果用户要实现 topic 全局的消息读取顺序，就只能通过让每个 consumer group 下只包含一个consumer 实例的方式来间接实现(或者也可以基于业务逻辑自己控制实现)。

独立消费者只有一个消费者实例单独执行消费操作。

### 构建 consumer
构造一个 consumer 需要以下6个步骤：
1. 构造一个 `Properties` 对象，至少指定 `bootstrap.servers` 、 `key.serializer` 和 `value.serializer` 和 `group.id` 的值。
2. 使用上一步创建的 `Properties` 实例构造 `KafkaConsumer` 对象
3. 调用 `KafkaConsumer.subscribe` 方法订阅感兴趣的 topic 列表。
4. 循环调用 `KafkaConsumer.pull` 方法获取封装为 `ConsumerRecord` 的 topic 信息。
5. 处理获取到的 `ConsumerRecord` 对象。
6. 关闭 `KafkaConsumer` 。

```
class Consumer implements Runnable{
    @Override
    public void run() {
        Properties consumerProp = new Properties();
        consumerProp.put("bootstrap.servers","[::1]:9093");
        consumerProp.put("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        consumerProp.put("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        consumerProp.put("group.id","test-group"+ Math.random());
        consumerProp.put("auto.offset.reset","earliest");
        KafkaConsumer<String,String> consumer = new KafkaConsumer<>(consumerProp);
        List topics = new ArrayList<String>();
        topics.add("quick-start");
        consumer.subscribe(topics);
        try {
            while (true) {
                ConsumerRecords<String, String> records = consumer.poll(100);
                for (ConsumerRecord record : records) {
                    System.out.println(record.key() + ":" + record.value());
                }
            }
        }finally {
            consumer.close();
        }
    }
}
```

### Consumer 的主要参数
1. `session.timeout.ms`
   Kafka的消费者协调者组件检测消费端（消费端宕机）失败的时间。
2. `max.poll.interval.ms`
   用于设置消息处理逻辑的最大时间。
3. `auto.commit.interval.ms`
   设置自动提交位移间隔时间，默认为 5 秒。
4. `auto.offset.reset`
   指定了无位移信息或位移越界（即consumer要消费的消息的位移不在当前消息日志的合理区间范围）时Kafka的应对策略。目前参数有三个值：
    + earliest：从最早的位移开始消费。不一定是0。
    + latest: 从最新位移处开始消费。
    + none
5. `enable.auto.commit`
   该参数指定 consumer 是否自动提交位移。如果设置为 false ，则需要用户在程序中手动提交位移。
6. `fetch.max.bytes`
   指定了 consumer 端单次获取数据的最大字节数。
7. `max.poll.records`
   该参数控制单次 poll 调用返回的最大消息数。
8. `heartbeat.interval.ms`
   该参数和
9.  `connections.max.idle.ms`
   Kafka 会定期地关闭空闲 socket 连接，这个参数指定空闲时间。如果超过空闲时间，那么 socket 连接会被关闭，下次要处理请求时，需要重新创建连接 broker 的 socket 连接。

### 位移
Offset 记录了 consumer 实例消费的消息的位置，Kafka 让 consumer 端保存 offset 。consumer 客户端需要定期地向 Kafka 集群汇报自己消费数据的进度，这一过程被称为**位移提交**。Kafka 内部有一个专门用来记录 consumer 端位移信息的 topic —— __consumer_offsets 。consumer 端需要为每个它要读取的分区保存消费进度，即分区中当前最新消费消息的位置 。该位置就被称为位移（ offset ） 。 consumer 需要定期地向 Kafka 提交自己的位置信息，实际上，这里的位移值通常是下一条待消费的消息的位置。假设 consumer 己经读取了某个分区中的第N条消息，那么它应该提交位移值为 N，因为位移是从 0 开始的，位移为 N 的消息是第 N+l 条消息 。 这样下次 consumer 重启时会从第 N+l 条消息开始消费。总而言之， offset 就是 consumer 端维护的位置信息 。

offset 对于 consumer 非常重要，因为它是实现消息交付语义保证（ message delivery semantic ）的基石 。 常见的 3 种消息交付语义保证如下。
+ 最多一次（ at most once ）处理语义：消息可能丢失，但不会被重复处理 。
+ 最少一次（ at least once ）处理语义：消息不会丢失，但可能被处理多次。
+ 精确一次（ exactly once ）处理语义：消息一定会被处理且只会被处理一次 。

consumer 提交位移的主要机制是通过向所属的 coordinator 发送位移提交请求来实现的 。每个位移提交请求都会往＿consumer_offsets 对应分区上追加写入一条消息 。 消息的 key 是group.id 、 topic 和分区的元组，而 value 就是位移值。 如果 consumer 为同一个 group 的同一个topic 分区提交了多次位移，那么一consumer_offsets 对应的分区上就会有若干条 key 相同但value 不同的消息，但显然我们只关心最新一次提交的那条消息。从某种程度来说，只有最新提交的位移值是有效的，其他消息包含的位移值其实都已经过期了 。 Kafka 通过压实（ compact)策略来处理这种消息使用模式。

默认情况下， consumer 是自动提交位移的，自动提交间隔是 5 秒。这就是说若不做特定的设置 ， consumer 程序在后台自动提交位移。通过设置 `auto.commit.interval.ms` 参数可以控制自动提交的间隔。

自动位移提交的优势是降低了用户的开发成本使得用户不必亲自处理位移提交：劣势是用户不能细粒度地处理位移的提交，特别是在有较强的精确一次处理语义时。在这种情况下，用户可以使用手动位移提交。

设置使用手动提交位移非常简单，仅仅需要在构建 KafkaConsumer 时设置 enable.auto.comrnit=false ，然后调用 comrnitSync 或commitAsync 方法即可。

### 重平衡
consumer group 的 rebalance 本质上是一组协议，它规定了一个 consumer group 是如何达成一致来分配订阅 topic 的所有分区的 。 topic 的每个分区只会分配给组内的一个 consumer 实例。

Kafka内置的一个全新的组协调协议（ group coordination protocol)负责重平衡 。对于每个组而言， Kafka 的某个broker 会被选举为组协调者（ group coordinator) o coordinator 负责对组的状态进行管理，它的主要职责就是当新成员到达时促成组内所有成员达成新的分区分配方案，即 coordinator 负责对组执行 rebalance 操作。

组 rebalance 触发的条件有以下 3 个：
+ 组成员发生变更，比如新 consumer 加入组，或己有 consumer 主动离开组，再或是己有 consumer 崩溃时则触发 rebalance 。
+ 组订阅 topic 数发生变更，比如使用基于正则表达式的订阅，当匹配正则表达式的新topic 被创建时则会触发 rebalance，因为这会导致消费的分区数发生变化 。
+ 组订阅 topic 的分区数发生变更，比如使用命令行脚本增加了订阅 topic 的分区数 。

分区分配策略有3种，分别是：
1. range 策略，默认策略，可以使用`partition.assignment.strategy`参数修改
2. round-robin 策略
3. sticky 策略
   
range 策略主要是基于范围的思想。它将单个 topic 的所有分区按照顺序排列，然后把这些分区划分成固定大小的分区段井依次分配给每个 consumer; round-robin 策略则会把所有 topic 的所有分区顺序摆开，然后轮询式地分配给各个 consumer 。 最新发布的 sticky 策略有效地避免了上述两种策略完全无视历史分配方案的缺陷，采用了“有黏性”的策略对所有 consumer 实例进行分配，可以规避极端情况下的数据倾斜并且在两次 rebalance I司最大限度地维持了之前的分配方案 。

另外 Kafka 支持自定义的分配策略，用户可以创建自己的 consumer 分配器（ assignor ） 。

### 消息交付语义
前面提到了3种消息交付语义：
+ 最多一次（ at most once ）处理语义：消息可能丢失，但不会被重复处理 。
+ 最少一次（ at least once ）处理语义：消息不会丢失，但可能被处理多次。
+ 精确一次（ exactly once ）处理语义：消息一定会被处理且只会被处理一次 。

对 producer 而言， Kafka 引入己提交消息 committed message ）的概念 。一旦消息被成功地提交到日志文件，只要至少存在一个可用的包含该消息的副本，那么这条消息就永远不会丢失。在 0.11.0 .0 版本之前， Kafka producer 默认提供的是 at least once 语义 。

设想这样的一个场景，当 producer 向 broker 发起新消息后，分区 leader 副本所在的 broker 成功地将该消息写入本地磁盘，然后发送响应给 producer。 此时假设网络出现故障导致该响应没有发送成功，那么未接到响应的 producer 会认为该消息请求失败从而开启重试操作。若重试后网络恢复正常，那么显然同一条消息被写入到日志两次。在比较极端的情况下，同一条消息可能会被发送多次。

0.11.0.0 版本引入了幂等性 producer 表示它的发送操作是幂等的 。 瞬时的发送错误可能导致 producer 端出现重试，同一条消息被 producer 发送多次，但在 broker 端这条消息只会被写入日志一次。 对于单个 topic 分区而言 ，这种 producer 提供的幂等性消除了各种错误导致的重复消息。 如果要启用幂等 性 producer 以及获取其提供的 EOS 语义，用户需要显式地设置 producer 端的新参数 `enable.idempotence` 为 true 。

幂等性 producer 的设计思路类似于 TCP 的工作方式，发送到 broker 端的每批消息都会被赋予一个序列号（ sequence number ）用于消息去重。但是和 TCP 不同的是，这个序列号不会被丢弃，相反 Kafka 会把它们保存在底层日志中，这样即使分区的 leader 副本挂掉，新选出来的 leader broker 也能执行消息去重工作。 保存序列号只需要额外几字节，因此整体上对 Kafka消息保存开销的影响并不大。除了序列号， Kafka 还会为每个 producer 实例分配一个 producer id （下称 PID ）。 producer在初始化时必须分配一个 PID 。 PID 分配的过程对用户来说是完全透明的，因此不会为用户所见。 消息要被发送到的每个分区都有对应的序列号值，它们总是从 0 开始并且严格单调增加 。对于 PID 、分区和序列号的关系，用户可以设想一个 Map, key 就是 （PID ，分区号）, value 就是序列号。即每对（PID ，分区号）都有对应的序列号值。若发送消息的序列号小于或等于broker 端保存的序列号，那么 broker 会拒绝这条消息的写入操作 。

<center>
	<img src="pics/kafka-idempotent.png" width="40%">
</center>

对事务的支持是 Kafka 实现 EOS 的第二个利器。
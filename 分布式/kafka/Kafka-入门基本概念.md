# Kafka入门
{docsify-updated}

### 消息系统的模型
最常见的两种消息泛型：
1. 消息队列模型  
    基于消息队列提供的消息服务，用于进程间通信以及线程间通信。该模型定义了消息队列、发送者和接收者，提供了一种点对点的消息传递方式，发送者发送每条消息到队列的指定位置，接收者从指定位置获取消息。每条消息由一个发送者发送出来，且只会被一个消费者消费：发送者和消费者是一对一的关系。
2. 发布/订阅模型  
    有一个主题概念的，主题可以理解为逻辑语义相近的消息的容器。这种模型一般将发送者称为发布者，将消费者称为订阅者。发布者将消息发布到指定的主题 topic 中，所有订阅了该 topic 的订阅者都可以收到该 topic 下的所有消息。

### Kafka 概要设计
1. 高吞吐量/低时延  
    Kafka 虽然会持久化所有数据到磁盘，但本质上每次写入操作其实都只是把数据写入到操作系统的页缓存中，然后由操作系统自行决定什么时候把也缓存中的数据写回磁盘。这样有三个优势：
    1. 操作系统页缓存是在内存中分配的，所以消息写入的速度非常快
    2. Kafka 不直接与底层的文件系统打交道。所有繁琐的IO操作都交由操作系统来处理。
    3. Kafka 写入操作用追加写入的方式（顺序写磁盘），避免了磁盘随机写操作。
  
Kafka 设计时采用追加写入的消息的方式，即智能在日志文件的末尾追加写入新的消息，且不允许修改已写入的消息，顺序写磁盘的速度甚至能匹配内存读写速度，所以Kafka消息发送的吞吐量是很高的。
读取消息时，同样也会首先尝试从OS也缓存中读取，如果命中便把消息经也缓存直接发送到网络的Socket上。这就是大名鼎鼎的零拷贝技术。
<center><img src="pics/no_zero_copy.png" alt="" width="40%"></center>
不使用零拷贝时，数据会进行多次拷贝，并且涉及到内核态与用户态的上下文切换，CPU开销很大。
<center><img src="pics/zero_copy.png" alt="" width="40%"></center>
Linux 提供的 sendfile 系统调用实现了这种零拷贝技术，Kafka的消息消费机制就是使用的该调用，严格来说是通过 Java 的 FileChannel.transferTo 方法实现的。

  总结起来，Kafka依靠以下4点达到高吞吐量、低延时的设计目标：
   1. 大量使用系统页缓存，内存操作速度高且命中率高
   2. Kafka不直接参与IO操作
   3. 采用追加写入的方式避免了磁盘随机读写，大幅提高磁盘写入速度
   4. 采用零拷贝技术使得网络IO速度大幅提升

2. 消息持久化  
    Kafka会持久化消息到磁盘，这样做的好处如下
        1. 解耦消息发送方与消息消费方，发送方只要将消息发送给Kafka服务保存即可。
        2. 实现灵活的消息处理，消息持久化能支持消息的重复处理，及重新处理一次之前的某个消息。

3. 负载均衡和故障转移  
    Kafka通过智能化的分区领导者选举来实现负载均衡。故障转移通过以“心跳”机制来实现的，只要服务器与备份服务器之间的心跳无法维持或者主服务器注册到服务中心的会话过期了，那么就认为主服务器已经无法正常运行，集群会自动启动某个备份服务器来代替主服务器的工作。

4. 伸缩性  
    阻碍线性扩容的一个很常见的因素就是状态的保存。无论哪类分布式系统，集群中的每台服务器一定会维护很多内部状态。如果服务器自身来保存这些状态信息，则必须要处理一致性问题。相反，如果服务器是无状态的，状态的保存和管理交于专门的协调服务来做，那么整个集群的服务器之间就无需繁重的状态共享，极大地降低了维护复杂度。
    Kafka使用 Zookeeper 来管理服务器状态，Kafka 服务器只保存很轻量级的内部状态。

### Kafka基本概念与术语
1. 消息  
2. topic 和 partition  
    topic是一个逻辑概念，代表了一类消息，也可以理解为一类消息的容器。topic 通常会被多个消费者订阅，因此出于性能的考虑，Kafka并不是 topic-message 的两级结构，而是采用了 topic-partition-message 的三级结构来分散负载。本质上说每个 topic 都由若干个 partition组成，每个partition是不可修改的有序消息队列（消息日志）。每个partition都有自己的专属partition编号，通常从0开始。用户能对partition唯一能做的就是在序列尾部追加写入消息。每个消息都有一个序号，该序号被称为位移（offset）。<topic,partition,offset> 能唯一确认一条消息。
    <center><img src="pics/topic-partition.png" alt="" width="40%"></center>

3. offset  
   topic下的partition中每个消息都有一个序号，该序号称为 offset。另外，消费端在消费消息时也有一个 offset 的概念，指向了当前消费到第几个消息了，通常该位移会随着消费进度不断往前移动。

4. replica  
   replica 就是 Kafka的容错备份机制，简单来说就是备份多份日志，备份的基本单位是 partition，也就是说会针对整个partition进行备份而不是一条消息，这些备份在 Kafka 中被称为副本（replica），他们存在的唯一目的就是为了防止数据丢失。副本分为两类：领导者副本（leader replica）和追随者副本（follower replica)。follower replica 是不能提供服务给客户端的，也就是说客户端的读取和写入请求不会route 到follower replica。它只会被动地从 leader replica 中获取数据，一旦 leader replica 所在的 broker 宕机， Kafka 会从剩余的 replica 中选举出新的 leader 继续提供服务。

5. leader 和 follower  
   Kafka保证同一个 partition 的多个 replica 一定不会分配到同一个 broker 上。
    <center><img src="pics/leader-follower.png" alt="" width="40%"></center>

6. ISR  
   ISR 的全称是 in-ync replica，就是与 leader replica保持同步的 replica 的集合，包括 leader replica。Kafka 为 partition 动态维护一各 replica 集合。集合中所有 replica 保存的消息都与 leader replica 保持同步状态，只有这个集合中的 replica 才能被选举为 leader，也只有该集合中的所有 replica 都接收到了同一条消息， Kafka 才会将该消息置为已提交状态，即认为该消息发送成功。不过这点可以通过配置修改。
   
### Kafka使用场景
1. 消息传输
2. 网站行为日志追踪
3. 设计数据收集
4. 日志收集
5. Event Sourcing
6. 流式处理
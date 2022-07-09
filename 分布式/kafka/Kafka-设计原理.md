# Kafka-设计原理
{docsify-updated}


#### 副本与 ISR 
一个 Kafka 分区本质上是一个备份日志，利用多个相同的备份共同提供冗余来保证系统的高可用性。这些备份在 Kafka 中称为副本。kafka 把副本均匀地分配到 broker 上，并从这些副本中挑选一个作为 leader 对外服务，其余副本称为 follower 副本，follower 被动地向 leader 副本请求数据从而保持与 leader 副本的同步。但是，在同步的过程中，有些 follower 因为各种原因（网络延迟/故障或者系统bug等）会落后 leader 的进度。Kafka 把那些与 leader 副本保持一致的副本集合称为 ISR （in-sync replicas)，包括leader副本本身也在ISR 中。

当producer 提交一条消息到到分区中时，只有 ISR 集合中所有分区副本都接收到并写入了，才被视为 “已提交” 状态。由此可见，若 ISR 包含 N个副本，则可以允许 N-1 个副本崩溃而不丢失消息。另外，当 leader 因为一些原因不能提供服务时，只有 ISR 中follower 副本才有资格去竞争当选新的 leader 。
### Redis 分布式集群
{docsify-updated}

- [Redis 分布式集群](#redis-分布式集群)
	- [Redis 集群的数据分片](#redis-集群的数据分片)
	- [Redis 集群的主从复制模型](#redis-集群的主从复制模型)
	- [搭建集群](#搭建集群)
		- [手动搭建](#手动搭建)
		- [使用Redis自带脚本创建](#使用redis自带脚本创建)
	- [Redis 一致性保证](#redis-一致性保证)


Redis 支持一主多从的主从复制和集群分片的组合模式。

#### Redis 集群的数据分片
Redis 集群没有使用一致性hash, 而是引入了哈希槽的概念.

Redis 集群有16384个哈希槽,每个key通过CRC16校验后对16384取模来决定放置哪个槽.集群的每个节点负责一部分hash槽,举个例子,比如当前集群有3个节点,那么:

节点 A 包含 0 到 5500号哈希槽.
节点 B 包含5501 到 11000 号哈希槽.
节点 C 包含11001 到 16384号哈希槽.
这种结构很容易添加或者删除节点. 比如如果我想新添加个节点D, 我需要从节点 A, B, C中得部分槽到D上. 如果我想移除节点A,需要将A中的槽移到B和C节点上,然后将没有任何槽的A节点从集群中移除即可. 由于从一个节点将哈希槽移动到另一个节点并不会停止服务,所以无论添加删除或者改变某个节点的哈希槽的数量都不会造成集群不可用的状态.

#### Redis 集群的主从复制模型
为了使在部分节点失败或者大部分节点无法通信的情况下集群仍然可用，所以集群使用了主从复制模型,每个主节点都会有N-1个复制品.

**在我们例子中具有A，B，C三个节点的集群,在没有复制模型的情况下,如果节点B失败了，那么整个集群就会以为缺少5501-11000这个范围的槽而不可用。**
然而如果在集群创建的时候（或者过一段时间）我们为每个节点添加一个从节点A1，B1，C1,那么整个集群便有三个master节点和三个slave节点组成，这样在节点B失败后，集群便会选举B1为新的主节点继续服务，整个集群便不会因为槽找不到而不可用了

不过当B和B1 都失败后，集群是不可用的.

#### 搭建集群

##### 手动搭建

1. 准备节点

   Redis集群一般由多个节点组成，节点数量至少为6个才能保证组成完整高可用的集群。每个节点需要开启配置 `cluster-enabled yes`，让Redis运行在集群模式下。建议为集群内所有节点统一目录，一般划分三个目录：conf、data、log，分别存放配置、数据和日志相关文件。把6个节点配置统一放在conf目录下，命名规则为 redis-port.conf ,集群相关配置如下：

   ```
   bind 127.0.0.1
   port 6379
   tcp-backlog 511
   loglevel notice
   logfile "log/node-6379.log"
   cluster-enabled yes
   \# 节点超时时间，单位毫秒
   cluster-node-timeout 15000
   \# 集群内部配置文件
   cluster-config-file "nodes-6379.conf"
   ```

   配置文件创建好后，分别启动6个实例：

   `redis-server conf\redis-6379.conf`

   实例启动后，可以用 redis-cli 连接到任意一台机器，然后执行：`cluster nodes`，会发现只有一台机器，因为此时6台实例之间并不知道对方。

2. 节点握手

   节点握手是指一批运行在集群模式下的节点通过Gossip协议彼此通信，达到感知对方的过程。节点握手是集群彼此通信的第一步，由客户端发起命令：`cluster meet {ip} {port}`

   分别执行上述命令将6台机器加入到集群后，集群还不能正常工作，这时集群处于下线状态，所有的数据读写都被禁止，可以使用 `cluster info` 命令查看集群的当前状态：

   ```
   cluster_state:fail
   cluster_slots_assigned:0
   cluster_slots_ok:0
   cluster_slots_pfail:0
   cluster_slots_fail:0
   cluster_known_nodes:6
   cluster_size:0
   cluster_current_epoch:1
   cluster_my_epoch:1
   cluster_stats_messages_ping_sent:101
   cluster_stats_messages_meet_sent:5
   cluster_stats_messages_sent:106
   cluster_stats_messages_pong_received:106
   cluster_stats_messages_received:106
   ```

   因为此时还没有为各个节点分配槽，所以现在集群还是不可用的。

3. 分配槽

   Redis集群把所有的数据映射到16384个槽中。每个key会映射为一个固定的槽，只有当节点分配了槽，才能响应和这些槽关联的键命令。通过 `cluster addslots` 命令为节点分配槽。这里利用bash特性批量设置槽（slots），命令如下：

   ```
   redis-cli -h 127.0.0.1 -p 6379 cluster addslots {0...5461}
   windows:FOR /L %i IN (0,1,5461) DO ( redis-cli.exe -h 127.0.0.1 -p 6379 CLUSTER ADDSLOTS %i )
   redis-cli -h 127.0.0.1 -p 6380 cluster addslots {5462...10922}
   windows:FOR /L %i IN (5462,1,10922) DO ( redis-cli.exe -h 127.0.0.1 -p 6380 CLUSTER ADDSLOTS %i )
   redis-cli -h 127.0.0.1 -p 6381 cluster addslots {10923...16383}
   windows:FOR /L %i IN (10923,1,16383) DO ( redis-cli.exe -h 127.0.0.1 -p 6381 CLUSTER ADDSLOTS %i )
   ```
   
   分配好槽以后，整个集群就是可用的了：
   
   ```
   127.0.0.1:6379> cluster info
   cluster_state:ok
   cluster_slots_assigned:16384
   cluster_slots_ok:16384
   cluster_slots_pfail:0
   cluster_slots_fail:0
   cluster_known_nodes:6
   cluster_size:3
   cluster_current_epoch:1
   cluster_my_epoch:1
   cluster_stats_messages_ping_sent:10253
   cluster_stats_messages_sent:10253
   cluster_stats_messages_pong_received:10017
   cluster_stats_messages_received:10017
   ```
   
   目前还有三个节点没有使用，作为一个完整的集群，每个负责处理槽的节点应该具有从节点，保证当它出现故障时可以自动进行故障转移。集群模式下，Reids节点角色分为主节点和从节点。首次启动的节点和被分配槽的节点都是主节点，从节点负责复制主节点槽信息和相关的数据。使用`cluster replicate {nodeId}`命令让一个节点成为从节点。其中命令执行必须在对应的从节点上执行，`nodeId` 是要复制主节点的节点ID，命令如下：
   
   ```
   redis-cli -h 127.0.0.1 -p 6382 cluster replicate a62b0061a541872d5c41e75efe987283aed167f6
   redis-cli -h 127.0.0.1 -p 6383 cluster replicate a935e92708b09b6ad2f4ae10c433be519c4ecfd0
   redis-cli -h 127.0.0.1 -p 6384 cluster replicate c94dde54d0f6aa124356d8a58c6be0a8c4ae8058
   ```
   
   执行完后，就会发现此时的集群是3主3从的集群了：
   
   ```
   127.0.0.1:6379> cluster nodes
   55ab7f5801001c9c71809a86b394f4fd029e87a2 127.0.0.1:6382@16382 slave a62b0061a541872d5c41e75efe987283aed167f6 0 1612365584000 4 connected
   a9f663f870d15f1d2c13665d49e4b8a877ce7467 127.0.0.1:6384@16384 slave c94dde54d0f6aa124356d8a58c6be0a8c4ae8058 0 1612365582000 5 connected
   def8275ed31a1542892ff3444153476484f3934c 127.0.0.1:6383@16383 slave a935e92708b09b6ad2f4ae10c433be519c4ecfd0 0 1612365584293 2 connected
   a62b0061a541872d5c41e75efe987283aed167f6 127.0.0.1:6379@16379 myself,master - 0 1612365583000 1 connected 0-5460
   c94dde54d0f6aa124356d8a58c6be0a8c4ae8058 127.0.0.1:6381@16381 master - 0 1612365585386 3 connected 10923-16383
   a935e92708b09b6ad2f4ae10c433be519c4ecfd0 127.0.0.1:6380@16380 master - 0 1612365583000 2 connected 5461-10922
   ```
   
   使用 `redis-cli -c` 参数连接到集群中任意一台机器上，然后使用 `get/set` 命令存取数据，这样会用 key 计算 hash 然后算出对应的槽，客户端也会自动重定向到槽所对应的节点上存取数据。
   `redis-cli -c --cluster call 127.0.0.1:6379 keys *` 查看集群中的所有key

##### 使用Redis自带脚本创建
下载官方安装包，解压后，执行 `make install` 安装 redis。
然后到 utils/create-cluster 目录下执行 ：
```
1. create-cluster start
2. create-cluster create
3. create-cluster stop
```
**如果是用 yum/apt 安装的redis，又想用官方的 utils/create-cluster 创建集群，可以修改 utils/create-cluster 脚本，将所有的带路径的 redis-server/redis-cli 换成不带前缀的。**

[create-cluster脚本](./create-cluster.md)

#### Redis 一致性保证
Redis 并不能保证数据的强一致性. 这意味这在实际中集群在特定的条件下可能会丢失写操作.

第一个原因是因为集群是用了异步复制. 写操作过程:

客户端向主节点B写入一条命令.
主节点B向客户端回复命令状态.
主节点将写操作复制给他得从节点 B1, B2 和 B3.
主节点对命令的复制工作发生在返回命令回复之后， 因为如果每次处理命令请求都需要等待复制操作完成的话， 那么主节点处理命令请求的速度将极大地降低 —— 我们必须在性能和一致性之间做出权衡。 注意：Redis 集群可能会在将来提供同步写的方法。 
Redis 集群另外一种可能会丢失命令的情况是集群出现了网络分区， 并且一个客户端与至少包括一个主节点在内的少数实例被孤立。

举个例子 假设集群包含 A 、 B 、 C 、 A1 、 B1 、 C1 六个节点， 其中 A 、B 、C 为主节点， A1 、B1 、C1 为A，B，C的从节点， 还有一个客户端 Z1 假设集群中发生网络分区，那么集群可能会分为两方，大部分的一方包含节点 A 、C 、A1 、B1 和 C1 ，小部分的一方则包含节点 B 和客户端 Z1 .

Z1仍然能够向主节点B中写入, 如果网络分区发生时间较短,那么集群将会继续正常运作,如果分区的时间足够让大部分的一方将B1选举为新的master，那么Z1写入B中得数据便丢失了.

注意， 在网络分裂出现期间， 客户端 Z1 可以向主节点 B 发送写命令的最大时间是有限制的， 这一时间限制称为节点超时时间（node timeout）， 是 Redis 集群的一个重要的配置选项：

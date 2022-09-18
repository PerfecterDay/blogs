# docker 网络

Docker 网络从覆盖范围可以分为单个 host 上的网络和跨宿主机的网络。

## 单宿主机的网络
Docker 安装时会自动在宿主机上创建三个网络，可以用 `docker network ls` 查看：

<center><img src="pics/docker-network.jpg" width="60%" style="inline"></center>

### none 网络
none 就是什么都没有的网络，挂在这个网络下的容器除了 lo 之外，没有其它任何网卡。容器创建时可以通过 `--network=none` 指定 none 网络。

### host 网络
连接到 host 网络的容器共享 host 主机的网络栈，容器的网络配置与宿主机的完全一样。可以通过 `--network=host` 指定 host 网络。在容器中可以看到宿主机所有的网卡。  
直接使用host 网络最大的好处就是性能。

### bridge 网络
Docker 安装时会默认创建一个命名为 docker0 的 Linux bridge。如果不指定 `--network` ，新创建的容器默认都会挂载到 docker0 上。

### 自定义网络
除了docker默认安装的三种网络，用户可以根据业务需要创建自定义网络。  
Docker提供三种自定义网络的驱动：bridge、overlay 和 macvlan 。overlay 和 macvlan 用于创建跨主机网络。

可以通过 bridge 创建类似于 docker 默认安装的 braidge 网络：  
`docker network create --driver bridge mynet`  
还可以通过 `--subnet`和`--gateway`参数指定创建的网络的子网和网关地址:
`docker network create --driver bridge --subnet=172.22.26.0/24 --gateway=172.22.26.1 mynet2`

### 容器间通信
容器之间可以通过 IP、Docker DNS Server 或 joined 容器三种方式通信。

1. IP通信
	两个容器要能通信必须要有属于同一个网络的网卡，具体做法就是在创建容器的时候通过 `--network=<network>` 指定特定网络，或者通过 `docker network connect <network> <container>` 将现有容器加入到指定网络。

2. Docker DNS Server
	通过IP进行通信有一个缺点，就是在部署容器之前可能无法确定容器的IP，这样必须要等容器部署之后在指定IP，会比较麻烦。docker daemon 实现了一个内嵌的 DNS Server，使容器能直接通过容器名通信，只要在创建容器时通过`--name=<container_name>`指定容器名即可。但是，Docker DNS 只能在自定义网络中使用，默认的bridge 网络是无法使用的。

3. Joined 容器
	Joined 容器非常特别，它能是两个或多个容器共享同一个网络栈、网卡和配置信息，使容器之间可以通过 127.0.0.1 直接通信。只要在创建容器时指定`--network=container:<container_name>`。  
	Joined 适用于容器之间希望通过 loopback 高效快速地通信或者希望监控其它容器的网络流量时。

### 容器与外部通信
1. 容器访问网络
<center><img src="pics/docker-communication.png" width="50%" style="inline"></center>

2. 外部访问容器内服务
	端口映射：`-p <host_ip>:<host_port>:<container_port>`
	每一个映射的端口，host 都会启动一个 docker-proxy 进程来访问容器的流量。


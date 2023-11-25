## k8s-Service
{docsify-updated}

- [k8s-Service](#k8s-service)
	- [Service 创建](#service-创建)
	- [Service IP 原理](#service-ip-原理)
	- [外网如何访问 Service](#外网如何访问-service)


我们不应该期望 Kubernetes Pod 是健壮的，而是要假设 Pod 中的容器很可能因为各种原因发生故障而死掉。Deployment 等 controller 会通过动态创建和销毁 Pod 来保证应用整体的健壮性。换句话说，Pod 是脆弱的，但应用是健壮的。  
每个 Pod 都有自己的 IP 地址。当 controller 用新 Pod 替代发生故障的 Pod 时，新 Pod 会分配到新的 IP 地址。这样就产生了一个问题：  
如果一组 Pod 对外提供服务（比如 HTTP），它们的 IP 很有可能发生变化，那么客户端如何找到并访问这个服务呢？

Kubernetes 给出的解决方案是 Service。


### Service 创建
Kubernetes Service 从逻辑上代表了一组 Pod，具体是哪些 Pod 则是由selector 指明挑选那些 label 的容器来服务。Service 有自己 IP，而且这个 IP 是不变的。客户端只需要访问 Service 的 IP，Kubernetes 则负责建立和维护 Service 与 Pod 的映射关系。无论后端 Pod 如何变化，对客户端不会有任何影响，因为 Service 没有变。


### Service IP 原理
Service Cluster IP 是一个虚拟 IP，是由 Kubernetes 节点上的 iptables 规则管理的。  
iptables 将访问 Service 的流量转发到后端 Pod，而且使用类似轮询的负载均衡策略。Cluster 的每一个节点都配置了相同的 iptables 规则，这样就确保了整个 Cluster 都能够通过 Service 的 Cluster IP 访问 Service。

在 Cluster 中，除了可以通过 Cluster IP 访问 Service，Kubernetes 还提供了更为方便的 DNS 访问。
k8s 部署时会默认安装 kube-dns 组件。
kube-dns 是一个 DNS 服务器。每当有新的 Service 被创建，kube-dns 会添加该 Service 的 DNS 记录。Cluster 中的 Pod 可以通过 <SERVICE_NAME>.<NAMESPACE_NAME> 访问 Service。

通过 namespace: kube-public 指定资源所属的 namespace。多个资源可以在一个 YAML 文件中定义，用 --- 分割

### 外网如何访问 Service
除了 Cluster 内部可以访问 Service，很多情况我们也希望应用的 Service 能够暴露给 Cluster 外部。Kubernetes 提供了多种类型的 Service，默认是 ClusterIP。

1. ClusterIP   
	Service 通过 Cluster 内部的 IP 对外提供服务，只有 Cluster 内的节点和 Pod 可访问，这是默认的 Service 类型，前面实验中的 Service 都是 ClusterIP。

2. NodePort   
	Service 通过 Cluster 节点的静态端口对外提供服务。Cluster 外部可以通过 `<NodeIP>:<NodePort>` 访问 Service。

3. LoadBalancer   
	Service 利用 cloud provider 特有的 load balancer 对外提供服务，cloud provider 负责将 load balancer 的流量导向 Service。目前支持的 cloud provider 有 GCP、AWS、Azur 等。

4. ExternalName  
	将服务映射到 externalName 字段的内容（例如，映射到主机名 api.foo.bar.example）。 该映射将集群的 DNS 服务器配置为返回具有该外部主机名值的 CNAME 记录。 无需创建任何类型代理
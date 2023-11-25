## k8s DaemonSet
{docsify-updated}

- [k8s DaemonSet](#k8s-daemonset)

Deployment 部署的副本 Pod 会分布在各个 Node 上，每个 Node 都可能运行好几个副本。DaemonSet 的不同之处在于：**每个 Node 上最多只能运行一个副本**。

DaemonSet 的典型应用场景有：
+ 在集群的每个节点上运行存储 Daemon，比如 glusterd 或 ceph。
+ 在每个节点上运行日志收集 Daemon，比如 flunentd 或 logstash。
+ 在每个节点上运行监控 Daemon，比如 Prometheus Node Exporter 或 collectd。

其实 Kubernetes 自己就在用 DaemonSet 运行系统组件。执行如下命令：
`kubectl get daemonset --namespace=kube-system`
可以看到 DaemonSet kube-flannel-ds 和 kube-proxy 分别负责在每个节点上运行 flannel 和 kube-proxy 组件。 

1. DaemonSet 配置文件的语法和结构与 Deployment 几乎完全一样，只是将 kind 设为 DaemonSet。
2. hostNetwork 指定 Pod 直接使用的是 Node 的网络，相当于 docker run --network=host。考虑到 flannel 需要为集群提供网络连接，这个要求是合理的。
3. containers 定义了运行 flannel 服务的两个容器。

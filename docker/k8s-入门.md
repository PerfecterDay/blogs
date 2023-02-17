## k8s hello world
{docsify-updated}

- [k8s hello world](#k8s-hello-world)
	- [重要概念](#重要概念)
	- [工作节点组件](#工作节点组件)
	- [K8S部署示意图](#k8s部署示意图)
	- [Minikube安装运行k8s](#minikube安装运行k8s)


### 重要概念
1. Cluster  
	Cluster 是计算、存储和网络资源的集合，Kubernetes 利用这些资源运行各种基于容器的 应用。
2. Master  
	Master 是Cluster 的大脑，它的主要职责是调度，即决定将应用放在哪里运行。Master 运 行Linux 操作系统，可以是物理机或者虚拟机。为了实现高可用，可以运行多个Master。
3. Node
	Node 的职责是运行容器应用。Node 由 Master 管理，Node 负责监控并汇报容器的状态， 同时根据Master 的要求管理容器的生命周期。Node 运行在Linux操作系统上，可以是物理 机或者是虛拟机。
4. Pod  
	Pod 是 Kubernetes 的最小工作单元。每个 Pod 包含 一个或多个容器。Pod 中的容器会 作为一个整体被Master 调度到一个Node 上运行。
5. Controller  
	K8S 通过 Controller 来管理 Pod。Controller 中定义了 Pod 的部署特性，比如有几个副本、在什么样的 Node 上运行等。K8S 提供了多种 Controller：
	+ Deployment：可以管理 Pod 的多个副本并确保 Pod 按照期望的状态运行
	+ ReplicaSet：实现了 Pod 的多副本管理。使用 Deployment 会自动创建 ReplicaSet，Deployment 通过 ReplicaSet 来管理 Pod 的多个副本，一般不直接使用 ReplicaSet。
	+ DaemonSet：用于每隔 Node 最多只运行一个 Pod 副本的场景。DaemonSet 通常用于运行 daemon。
	+ StatefuleSet：能够保证 Pod 的每个副本在整个生命周期中是不变的，而其他 Controller 不提供这个功能。当某个 Pod 发生故障需要删除并重新启动时，Pod 的名称会发生变化，同时 StatefuleSet 会保证副本按照固定顺序启动、更新或者删除。
	+ Job：用于运行结束就删除的应用，而其他 Controller 中的 Pod 通常是长期持续运行。
6. Service  
	Deployment 可以部署多个副本，每个Pod都有自己的IP，外界如何访问这些Pod的服务呢？通过IP吗？Pod会被频繁的重启和销毁重建，他们的IP会发生变化，用IP不能动态适应这种情形。  
	K8S Service 定义了外界访问一组特定 Pod 的方式。Service 有自己的 IP 和端口，Service 为 Pod 提供了负载均衡。  
	K8S 运行容器（Pod）与访问容器（Pod）这两个任分别由 Controller 和 Service 执行。
7. Namespace  
	可以将物理的 Cluster 逻辑上划分成多个虚拟 Cluster，每个 Cluster 就是一个 Namespace、不同的 Namespace 里的资源是完全隔离的。K8S 默认创建了两个 Namespace。
	+ default：创建资源时如果不指定，将被放到这个 Namespace 中
	+ kube-system：K8S 自己创建的系统资源将放到这个 Namespace 中

<center><img src="pics/k8s-components.jpg" alt="" width="60%"></center>

### 工作节点组件

1. kubelet  
	一个在集群中每个节点上运行的代理。它保证容器都运行在 Pod 中。  
	kubelet 接收一组通过各类机制提供给它的 PodSpecs，确保这些 PodSpecs 中描述的容器处于运行状态且健康。kubelet 不会管理不是由 Kubernetes 创建的容器。

2. kube-proxy  
	kube-proxy 是集群中每个节点上运行的网络代理,实现 Kubernetes Service 概念的一部分。  
	kube-proxy 维护节点上的网络规则。这些网络规则允许从集群内部或外部的网络会话与 Pod 进行网络通信。  
	如果有 kube-proxy 可用，它将使用操作系统数据包过滤层。否则，kube-proxy 会转发流量本身。  

3. 容器运行环境(Container Runtime)  
	容器运行环境是负责运行容器的软件。  
	Kubernetes 支持多个容器运行环境: Docker、 containerd、cri-o、 rktlet 以及任何实现 Kubernetes CRI (容器运行环境接口)。

### K8S部署示意图
<center><img src="pics/k8s-demo.jpg" alt="" width="60%"></center>

### Minikube安装运行k8s
1. 安装 Kubectl: `brew install kubectl`
2. 安装 Minikube: `brew install minikube`
3. 执行下列语句以使用本地docker镜像：`eval $(minikube docker-env)`
4. 启动 Minikube 并创建一个集群：`minikube start --driver=docker`
5. 查看集群信息： `kubectl cluster-info`、`kubectl get nodes` 。
6. 使用名为 echoserver 的镜像创建一个 Kubernetes Deployment: `kubectl create deployment hello-nginx --image=nginx`,删除的话使用：`kubectl delete deployment hello-nginx`
7. 查看集群的 deployments信息：`kubectl get deployments`
8. 查看集群的pods及日志信息： `kubectl get pods`,`kubectl describe pods [pod名]`，`kubectl log [pod名字]`
9. 将其作为 Service 公开: `kubectl expose deployment hello-nginx --type=NodePort --port=8080`
10. 获取暴露 Service 的 URL 以查看 Service 的详细信息: `kubectl describe services hello-nginx`

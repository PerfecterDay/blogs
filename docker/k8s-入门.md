## k8s hello world
{docsify-updated}

- [k8s hello world](#k8s-hello-world)
	- [工作节点组件](#工作节点组件)
	- [K8S部署示意图](#k8s部署示意图)
	- [Minikube安装运行k8s](#minikube安装运行k8s)

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

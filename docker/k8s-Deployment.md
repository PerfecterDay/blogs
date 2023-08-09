## k8s Deployment
{docsify-updated}

前面我们已经了解到，Kubernetes 通过各种 Controller 来管理 Pod 的生命周期。为了满足不同业务场景，Kubernetes 开发了 Deployment、ReplicaSet、DaemonSet、StatefuleSet、Job 等多种 Controller。

### Deployment
Deployment 通过 ReplicaSet 来管理 Pod 的。

1. 用户通过 kubectl 创建 Deployment。
2. Deployment 创建 ReplicaSet。
3. ReplicaSet 创建 Pod。

<center><img src="pics/deployment.jpg" width=30%></center>
从上图也可以看出，对象的命名方式是：子对象的名字 = 父对象名字 + 随机字符串或数字。

### 创建资源的方式
Kubernetes 支持两种方式创建资源：
1. 用 kubectl 命令直接创建，比如：
	`kubectl run nginx-deployment --image=nginx:1.7.9 --replicas=2`
	在命令行中通过参数指定资源的属性。

2. 通过配置文件和 kubectl apply 创建，要完成前面同样的工作，可执行命令：
	`kubectl apply -f nginx.yml`
	nginx.yml 的内容为：
	```
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	name: nginx-deployment
	labels:
		app: nginx
	spec:
	replicas: 3
	selector:
		matchLabels:
		app: nginx
	template:
		metadata:
		labels:
			app: nginx
		spec:
		containers:
		- name: nginx
			image: nginx:1.14.2
			ports:
			- containerPort: 80
	```

+ apiVersion: 是当前配置格式的版本。
+ kind: 是要创建的资源类型，这里是 Deployment。
+ metadata: 是该资源的元数据，name 是必需的元数据项。
+ spec: 部分是该 Deployment 的规格说明。
+ replicas: 指明副本数量，默认为 1。
+ template: 定义 Pod 的模板，这是配置文件的重要部分。
+ metadata: 定义 Pod 的元数据，至少要定义一个 label。label 的 key 和 value 可以任意指定。
+ spec: 描述 Pod 的规格，此部分定义 Pod 中每一个容器的属性，name 和 image 是必需的。

**基于命令的方式**：
   + 简单直观快捷，上手快。
   + 适合临时测试或实验。

**基于配置文件的方式**：
   + 配置文件描述了 What，即应用最终要达到的状态。
   + 配置文件提供了创建资源的模板，能够重复部署。
   + 可以像管理代码一样管理部署。
   + 适合正式的、跨环境的、规模化部署。

这种方式要求熟悉配置文件的语法，有一定难度。
后面我们都将采用配置文件的方式，大家需要尽快熟悉和掌握。

+ `kubectl get deployment`: 查看部署的 deployment 资源
+ `kubectl get deployment [deploymentName]`：查看deployment的详细描述

### 伸缩
伸缩（Scale Up/Down）是指在线增加或减少 Pod 的副本数。
上述配置文件中的 Deployment nginx-deployment 初始是两个副本。只要修改配置文件中的 `replicas` 的数量并重新执行 `kubectl apply -f nginx.yml` 便可以伸缩副本数量。

### Failover
上一节我们有 3 个 nginx 副本分别运行在 k8s-node1 和 k8s-node2 上。现在模拟 k8s-node2 故障，关闭该节点。则在该节点上运行的 pod 状态会变为 Unknown 。
等待一段时间，Kubernetes 会检查到 k8s-node2 不可用，将 k8s-node2 上的 Pod 标记为 Unknown 状态，并在 k8s-node1 上新创建两个 Pod，维持总副本数为 3。
当 k8s-node2 恢复后，Unknown 的 Pod 会被删除，不过已经运行的 Pod 不会重新调度回 k8s-node2。

### 用 label 控制pod的位置——指定节点部署
默认配置下，Scheduler 会将 Pod 调度到所有可用的 Node。不过有些情况我们希望将 Pod 部署到指定的 Node，比如将有大量磁盘 I/O 的 Pod 部署到配置了 SSD 的 Node；或者 Pod 需要 GPU，需要运行在配置了 GPU 的节点上。

**Kubernetes 是通过 label 来实现这个功能的。**

label 是 key-value 对，各种资源都可以设置 label，灵活添加各种自定义属性。比如执行如下命令标注 k8s-node1 是配置了 SSD 的节点。  
`kubectl label node k8s-node1 disktype=ssd`
然后通过 `kubectl get node --show-labels` 查看节点的 label。  
disktype=ssd 已经成功添加到 k8s-node1，除了 disktype，Node 还有几个 Kubernetes 自己维护的 label。  
有了 disktype 这个自定义 label，接下来就可以指定将 Pod 部署到 k8s-node1。  
在 Pod 模板的 spec 里通过 `nodeSelector` 指定将此 Pod 部署到具有 label `disktype=ssd` 的 Node 上。  
删除lable : `kubectl label node k8s-node1 disktype-`



# k8s-Service
{docsify-updated}

- [k8s-Service](#k8s-service)
	- [Service 创建](#service-创建)
	- [Service IP 原理](#service-ip-原理)
	- [Service的种类](#service的种类)
		- [ClusterIP](#clusterip)
		- [NodePort](#nodeport)
		- [LoadBalancer](#loadbalancer)
		- [ExternalName](#externalname)


我们不应该期望 Kubernetes Pod 是健壮的，而是要假设 Pod 中的容器很可能因为各种原因发生故障而死掉。Deployment 等 controller 会通过动态创建和销毁 Pod 来保证应用整体的健壮性。换句话说，Pod 是脆弱的，但应用是健壮的。  
每个 Pod 都有自己的 IP 地址。当 controller 用新 Pod 替代发生故障的 Pod 时，新 Pod 会分配到新的 IP 地址。这样就产生了一个问题：  
如果一组 Pod 对外提供服务（比如 HTTP），它们的 IP 很有可能发生变化，那么客户端如何找到并访问这个服务呢？

Kubernetes 给出的解决方案是 Service。


## Service 创建
Kubernetes Service 从逻辑上代表了一组 Pod，具体是哪些 Pod，则是由selector 来筛选带有特定label 值的Pod。**Service 有自己 IP，而且这个 IP 是不变的**。客户端只需要访问 Service 的 IP，Kubernetes 则负责建立和维护 Service 与 Pod 的映射关系。无论后端 Pod 如何变化，对客户端不会有任何影响，因为 Service 没有变。

```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: '2023-02-08T06:22:48Z'
  labels:
    micrometer-prometheus-discovery: 'true'
  name: user-center-svc
  namespace: default
  resourceVersion: '92021626'
  uid: fef0d177-5e7e-4817-8f18-e8408b503410
spec:
  clusterIP: 192.168.162.123
  clusterIPs:
    - 192.168.162.123
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - name: user-center-svc-8913
      port: 8913
      protocol: TCP
      targetPort: 8913
    - name: user-center-svc-8091
      port: 8091 //service 端口
      protocol: TCP
      targetPort: 8091 //POD端口
  selector:
    app: user-center-service
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

## Service IP 原理
Service Cluster IP 是一个虚拟 IP，是由 Kubernetes 节点上的 iptables 规则管理的。  
iptables 将访问 Service 的流量转发到后端 Pod，而且使用类似轮询的负载均衡策略。Cluster 的每一个节点都配置了相同的 iptables 规则，这样就确保了整个 Cluster 都能够通过 Service 的 Cluster IP 访问 Service。

在 Cluster 中，除了可以通过 Cluster IP 访问 Service，Kubernetes 还提供了更为方便的 DNS 访问。
k8s 部署时会默认安装 kube-dns 组件。
kube-dns 是一个 DNS 服务器。每当有新的 Service 被创建，kube-dns 会添加该 Service 的 DNS 记录。Cluster 中的 Pod 可以通过 <SERVICE_NAME>.<NAMESPACE_NAME> 访问 Service。

通过 namespace: kube-public 指定资源所属的 namespace。多个资源可以在一个 YAML 文件中定义，用 --- 分割

## Service的种类
除了 Cluster 内部可以访问 Service，很多情况我们也希望应用的 Service 能够暴露给 Cluster 外部。Kubernetes 提供了多种类型的 Service，默认是 ClusterIP。

### ClusterIP
Service 通过 Cluster 内部的 IP 对外提供服务，只有 Cluster 内的节点和 Pod 可访问，这是默认的 Service 类型

### NodePort
Service 通过 Cluster 节点的静态端口对外提供服务。Cluster 外部可以通过 `<NodeIP>:<NodePort>` 访问 Service。

### LoadBalancer
Service 利用 cloud provider 特有的 load balancer 对外提供服务，cloud provider 负责将 load balancer 的流量导向 Service。目前支持的 cloud provider 有 GCP、AWS、Azur 等。
```
apiVersion: v1
kind: Service
metadata:
annotations:
	service.beta.kubernetes.io/alibaba-cloud-loadbalancer-cert-id: 5101273941756630_186597c33ea_-1637077474_-1736767758
	service.beta.kubernetes.io/alibaba-cloud-loadbalancer-spec: slb.s1.small
	service.beta.kubernetes.io/alicloud-loadbalancer-address-type: intranet
creationTimestamp: '2023-02-21T02:39:56Z'
finalizers:
	- service.k8s.alibaba/resources
labels:
	service.beta.kubernetes.io/hash: 7ae73a1e15324eb203b3c2d08789d5f91152c4b56ca39e8c59b8990b
	service.k8s.alibaba/loadbalancer-id: lb-3nsiwzexohm9i5208ktn3
managedFields:
	- apiVersion: v1
	fieldsType: FieldsV1
	fieldsV1:
		'f:metadata':
		'f:finalizers':
			.: {}
			'v:"service.k8s.alibaba/resources"': {}
	manager: cloud-controller-manager
	operation: Update
	time: '2023-02-21T02:39:56Z'
	- apiVersion: v1
	fieldsType: FieldsV1
	fieldsV1:
		'f:metadata':
		'f:labels':
			.: {}
			'f:service.beta.kubernetes.io/hash': {}
			'f:service.k8s.alibaba/loadbalancer-id': {}
		'f:status':
		'f:loadBalancer':
			'f:ingress': {}
	manager: cloud-controller-manager
	operation: Update
	subresource: status
	time: '2023-02-21T02:40:05Z'
	- apiVersion: v1
	fieldsType: FieldsV1
	fieldsV1:
		'f:metadata':
		'f:annotations':
			.: {}
			'f:service.beta.kubernetes.io/alibaba-cloud-loadbalancer-cert-id': {}
			'f:service.beta.kubernetes.io/alibaba-cloud-loadbalancer-spec': {}
			'f:service.beta.kubernetes.io/alicloud-loadbalancer-address-type': {}
		'f:spec':
		'f:allocateLoadBalancerNodePorts': {}
		'f:externalTrafficPolicy': {}
		'f:internalTrafficPolicy': {}
		'f:ipFamilyPolicy': {}
		'f:ports':
			.: {}
			'k:{"port":443,"protocol":"TCP"}':
			.: {}
			'f:name': {}
			'f:port': {}
			'f:protocol': {}
			'f:targetPort': {}
			'k:{"port":8000,"protocol":"TCP"}':
			.: {}
			'f:name': {}
			'f:port': {}
			'f:protocol': {}
			'f:targetPort': {}
		'f:selector': {}
		'f:sessionAffinity': {}
		'f:type': {}
	manager: ACK-Console Apache-HttpClient
	operation: Update
	time: '2024-01-04T02:45:53Z'
name: uc-idc-slb
namespace: default
resourceVersion: '157006083'
uid: 4f1781c9-7e7f-41a6-80f5-920f26d6ca69
spec:
allocateLoadBalancerNodePorts: true
clusterIP: 192.168.219.247
clusterIPs:
	- 192.168.219.247
externalTrafficPolicy: Local
healthCheckNodePort: 31615
internalTrafficPolicy: Cluster
ipFamilies:
	- IPv4
ipFamilyPolicy: SingleStack
ports:
	- name: uc-idc-slb-8000-10000
	nodePort: 31357
	port: 8000
	protocol: TCP
	targetPort: 10000
	- name: uc-idc-slb-443-10000
	nodePort: 32035
	port: 443
	protocol: TCP
	targetPort: 10000
selector:
	app: envoy-hk-idc
sessionAffinity: None
type: LoadBalancer
status:
loadBalancer:
	ingress:
	- ip: 10.4.152.180
```

### ExternalName  
ExternalName类型的Service将集群外部的域名映射到集群内部的Service上，例如将外部的数据库域名映射到集群内部的Service名，那么就能在集群内部通过Service名直接访问。  
该映射将集群的 DNS 服务器配置为返回具有该外部主机名值的 CNAME 记录。 无需创建任何类型代理。
# Consul

https://learn.hashicorp.com/tutorials/consul/docker-container-agents  
https://book-consul-guide.vnzmi.com/04_run_agent.html

查询集群成员：`docker exec badger consul members`

docker run -d -p 8500:8500 -p 8600:8600/udp --name=badger consul agent -server -ui -node=10.176.81.23 -bootstrap-expect=1 -client=0.0.0.0



docker run --name=fox consul agent -node=client-1 -join=172.17.0.3


docker exec <container_id> consul snapshot save backup.snap
docker cp <container_id>:backup.snap ./

1. docker run -d --name=dev-consul -e CONSUL_BIND_INTERFACE=eth0 consul  （假设IP是172.17.0.2）
2. docker run -d -e CONSUL_BIND_INTERFACE=eth0 consul agent -dev -join=172.17.0.2
3. docker run -d -e CONSUL_BIND_INTERFACE=eth0 consul agent -dev -join=172.17.0.2

添加 Agent：
docker run -d --net=host -e 'CONSUL_LOCAL_CONFIG={"leave_on_terminate": true}' consul agent -bind=<external ip> -retry-join=<root agent ip>



spring-cloud-consul-

ConsulAutoRegistration

ConsulDiscoveryClientConfiguration
            ConsulDiscoveryProperties
HeartbeatProperties

ConsulAutoServiceRegistrationListener
只有web应用时才会注册服务


### [HTTP API](https://developer.hashicorp.com/consul/api-docs/catalog#list-nodes-for-service)
查找所有注册的服务信息： `curl http://localhost:8500/v1/catalog/services`
查找所有注册的数据中心信息： `curl http://localhost:8500/v1/catalog/datacenters`
根据服务名查找某个具体服务信息： `curl http://localhost:8500/v1/catalog/service/{serviceName}`

`curl http://localhost:8500/v1/agent/checks`
查看处于 critical 状态的服务：`curl http://localhost:8500/v1/health/state/critical`

```
curl --header "X-Consul-Namespace: *" http://127.0.0.1:8500/v1/health/node/my-node

curl http://127.0.0.1:8500/v1/health/service/my-service?ns=default


curl http://127.0.0.1:8500/v1/agent/services

```



### Consul API
http://localhost:8100/actuator/health

### 问题
preferIpAddress ： 测试环境需要IP访问，主机名不通

重启consul 后，服务不能重新注册：
https://github.com/spring-cloud/spring-cloud-consul/issues/197
https://github.com/spring-cloud/spring-cloud-consul/pull/691
https://github.com/spring-cloud/spring-cloud-consul/issues/727

```
Resolved.
need to add the below 2 properties:
spring.cloud.consul.discovery.heartbeat.enabled= true
spring.cloud.consul.discovery.heartbeat.reregister-service-on-failure=true
```

### Helm安装 consul 集群
前提（安装Helm）:
1. curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
2. chmod 700 get_helm.sh
3. ./get_helm.sh

添加 consul repo:
1. helm repo add hashicorp https://helm.releases.hashicorp.com
2. helm install consul hashicorp/consul --set global.name=consul-cluster --set server.storage=2Gi --namespace consul

https://developer.hashicorp.com/consul/docs/k8s/installation/install

`helm install consul hashicorp/consul --set global.name=consul --set server.storage=2Gi --create-namespace --namespace consul`
`helm install consul-cluster hashicorp/consul --set global.name=consul --set server.storage=1Gi --set server.storageClass=alicloud-disk-available --namespace consul`

helm install consul hashicorp/consul --set global.name=consul-cluster --set server.storage=2Gi --namespace consul


存储声明 PVC ：
  name: data-consul-consul-cluster-server-0
  namespace: consul
  resourceVersion: '34177039'
  uid: 91fe6fc4-85c7-48b5-b3b1-8b1085294d43

要与存储卷 PV ：
	claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: data-consul-consul-cluster-server-0
    namespace: consul
    resourceVersion: '34168920'
    uid: 91fe6fc4-85c7-48b5-b3b1-8b1085294d43

保持一致
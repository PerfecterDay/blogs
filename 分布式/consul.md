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


ConsulAutoServiceRegistrationListener
只有web应用时才会注册服务


### HTTP API
查找所有注册的服务信息： `curl http://localhost:8500/v1/catalog/services`
根据服务名查找服务信息： `curl http://localhost:8500/v1/catalog/service/{serviceName}`

`curl http://localhost:8500/v1/agent/checks`
查看处于 critical 状态的服务：`curl http://localhost:8500/v1/health/state/critical`

### Consul API
http://localhost:8100/actuator/health
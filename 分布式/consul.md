# Consul

https://learn.hashicorp.com/tutorials/consul/docker-container-agents


查询集群成员：docker exec -t dev-consul consul members

docker run -d -p 8500:8500 -p 8600:8600/udp --name=badger consul agent -server -ui -node=server-1 -bootstrap-expect=1 -client=0.0.0.0


docker run --name=fox consul agent -node=client-1 -join=172.17.0.3

docker exec badger consul members

docker exec <container_id> consul snapshot save backup.snap
docker cp <container_id>:backup.snap ./

1. docker run -d --name=dev-consul -e CONSUL_BIND_INTERFACE=eth0 consul  （假设IP是172.17.0.2）
2. docker run -d -e CONSUL_BIND_INTERFACE=eth0 consul agent -dev -join=172.17.0.2
3. docker run -d -e CONSUL_BIND_INTERFACE=eth0 consul agent -dev -join=172.17.0.2

添加 Agent：
docker run -d --net=host -e 'CONSUL_LOCAL_CONFIG={"leave_on_terminate": true}' consul agent -bind=<external ip> -retry-join=<root agent ip>
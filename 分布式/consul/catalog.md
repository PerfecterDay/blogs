#  Catalog API

`/catalog`端点在Consul中注册和注销节点、服务和检查。Consul中节点和服务的注册与取消。`/catalog`不应该与`/agent`混淆，因为一些API方法看起来很相似。



## 注册实体

## 查询节点
GET /v1/catalog/nodes

## 查询服务列表
GET /v1/catalog/services

GET /v1/catalog/services?wait=2s&index=21424&token= HTTP/1.1

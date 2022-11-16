# grpc入门
{docsify-updated}

<center><img src="pics/grpc.svg"></center>

### 一个最简单的gRPC实例
1. 定义调用服务  
    搭建gRPC的第一步就是要定义gRPC调用服务的请求和响应类型，通常使用 protocol buffuers 的 .proto 文件来定义。
    首先，使用 `service`  关键字定义一个服务，这个服务类似于Java 中的接口，可以在里边定义一个或多个方法。
    ```
    service RouteGuide {
      ...
    }
    ```
    然后，可以在服务内部定义 rpc 方法以及请求和响应的类型。gRPC 支持4种 rpc 方法定义：
    + 简单方法定义
      ```
	  rpc GetFeature(Point) returns (Feature) {}
	  ```
    + server-side streaming RPC
      ```
	  rpc ListFeatures(Rectangle) returns (stream Feature) {}
	  ```
    + client-side streaming RPC
      ```
	  rpc RecordRoute(stream Point) returns (RouteSummary) {}
	  ```
    + bidirectional streaming RPC
      ```
	  rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
	  ```
	最后，要完成好 rpc 方法中的请求和响应类型的定义。
	```
	message Point {
		int32 latitude = 1;
		int32 longitude = 2;
	}
	message Feature{
		...
	}
	```

2. 生成客户端和服务器端代码
3. 服务实现

```
grpcurl --plaintext localhost:9090 list
grpcurl --plaintext localhost:9090 list GreeterService
grpcurl --plaintext -d '{"name": "aaa"}' localhost:9090 GreeterService.sayHello

发送ByteString 格式数据时，需要Base64编码：
grpcurl --plaintext -d '{"msgno":1001,"json":{"req":{ "req": [ { "code": "1645", "params": { "username": "700210", "custermer_code": "700210", "nulltoken": "", "tradetoken": "", "password": "G2222222", "language": "2" }, "params_ext": {} } ] }}}' 10.187.144.42:8101 grpc_client.Greeter.handle
                                |
                                |
                                V
grpcurl --plaintext -d '{"msgno":1001, "json": "eyAicmVxIjogWyB7ICJjb2RlIjogIjE2NDUiLCAicGFyYW1zIjogeyAidXNlcm5hbWUiOiAiNzAwMjEwIiwgImN1c3Rlcm1lcl9jb2RlIjogIjcwMDIxMCIsICJudWxsdG9rZW4iOiAiIiwgInRyYWRldG9rZW4iOiAiIiwgInBhc3N3b3JkIjogIkcyMjIyMjIyIiwgImxhbmd1YWdlIjogIjIiIH0sICJwYXJhbXNfZXh0Ijoge30gfSBdIH0="}' 10.187.144.42:8101 grpc_client.Greeter.handle

grpcurl --plaintext -d '{"msgno":1001, "json": "eyAicmVxIjogWyB7ICJjb2RlIjogIjE2NDUiLCAicGFyYW1zIjogeyAidXNlcm5hbWUiOiAiNzAwMjEwIiwgImN1c3Rlcm1lcl9jb2RlIjogIjcwMDIxMCIsICJudWxsdG9rZW4iOiAiIiwgInRyYWRldG9rZW4iOiAiIiwgInBhc3N3b3JkIjogIkcyMjIyMjIyIiwgImxhbmd1YWdlIjogIjIiIH0sICJwYXJhbXNfZXh0Ijoge30gfSBdIH0="}' 127.0.0.1:8101 grpc_client.Greeter.handle

grpcurl --plaintext -d '{"msgno":2, "json": ""}' 127.0.0.1:8101 grpc_client.Greeter.handle

grpcurl --plaintext -d '' localhost:8101 grpc_client.Greeter.handle
```

### 配置
1. 服务端配置
	参照 grpc-server-spring-boot-autoconfigure jar 包中的 `net.devh.boot.grpc.server.config.GrpcServerProperties` 类
2. 客户端配置
	参照 grpc-client-spring-boot-autoconfigure jar 包中的 `net.devh.boot.grpc.client.config.GrpcChannelsProperties` 类
3. @GrpcClient("myService") 注解为每一个 RPC 服务调用者注册一个名字，可以在配置文件中为各个调用者定义不同的配置。GLOBAL代表全局配置。



consul health check 和 GRPC 服务注册端口区分开：
```
spring:
  application:
    name: trade-center-service
  cloud:
    consul:
      host: consul
      port: 8500
      discovery:
        service-name: ${spring.application.name}
        register: true
        port: ${grpc.server.port}
        healthCheckUrl: "http://10.176.81.23:${server.port}"
        health-check-critical-timeout: 3m
        heartbeat:
          enable: true
```
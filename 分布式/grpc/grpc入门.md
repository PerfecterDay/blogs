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
```

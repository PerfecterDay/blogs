# Grpc API
{docsify-updated}

> https://grpc.github.io/grpc-java/javadoc/index.html

gRPC 基于远程过程调用的客户-服务器模型。 客户端创建一个与服务器相连的通道。 RPC 由客户端发起并发送给服务器，服务器再回复客户端。 当客户端和服务器发送完信息后，它们会半关闭各自的连接。 服务器一关闭，RPC 就完成了。

要发送 RPC，首先要使用 `ManagedChannelBuilder.forTarget(java.lang.String)` 创建一个通道。 使用自动生成的 Protobuf 存根时，存根类将有用于封装通道的构造函数。 这些构造函数包括 `newBlockingStub`、 `newStub`  和 `newFutureStub` ，你可以根据自己的设计来使用它们，存根是客户端与服务器交互的主要方式。

要接收 RPC，请使用 `ServerBuilder.forPort(int)` 创建一个服务器。 Protobuf 存根将包含一个名为 `AbstractFoo` 的抽象类，其中 Foo 是你的服务名称。 扩展这个类，并将它的实例传递给 `ServerBuilder.addService(io.grpc.ServerServiceDefinition)` 。 服务器构建完成后，调用 `Server.start()` 开始接受 RPC。

客户端和服务器都应使用自定义执行器。 gRPC 运行时包含一个默认执行器，可以方便测试和示例，但并不适合在生产环境中使用。 

客户端和服务器也可以使用 `shutdown` 方法优雅地关闭。 gRPC 还支持更多高级功能，如名称解析、负载平衡、双向流、健康检查等。 请参阅客户端和服务器构建器中的相关方法。 gRPC 的开发主要在 Github 上进行，网址是 `https://github.com/grpc/grpc-java` ，gRPC 团队欢迎贡献和错误报告。 如果你有关于 gRPC 的问题，也可以在 grpc-io 上查看邮件列表。

```
ServerBuilder builder =
    NettyServerBuilder.forPort(12345)
        .addService(ProtoReflectionService.newInstance())
        .addService(v3DiscoveryServer.getAggregatedDiscoveryServiceImpl())
        .addService(v3DiscoveryServer.getClusterDiscoveryServiceImpl())
        .addService(v3DiscoveryServer.getEndpointDiscoveryServiceImpl())
        .addService(v3DiscoveryServer.getListenerDiscoveryServiceImpl())
        .addService(v3DiscoveryServer.getRouteDiscoveryServiceImpl());

Server server = builder.build();

server.start();
```
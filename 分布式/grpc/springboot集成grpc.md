# Springboot 集成 grpc

### grpc-spring-boot-starter 配置
上述配置过程非常复杂，可以使用开源工具类帮助我们简化这些步骤。

```
<dependency>
	<groupId>net.devh</groupId>
	<artifactId>grpc-spring-boot-starter</artifactId>
</dependency>
```

1. 服务端配置
	参照 grpc-server-spring-boot-autoconfigure jar 包中的 `net.devh.boot.grpc.server.config.GrpcServerProperties` 类
2. 客户端配置
	参照 grpc-client-spring-boot-autoconfigure jar 包中的 `net.devh.boot.grpc.client.config.GrpcChannelsProperties` 类
3. @GrpcClient("myService") 注解为每一个 RPC 服务调用者注册一个名字，可以在配置文件中为各个调用者定义不同的配置。GLOBAL代表全局配置。
4. @GrpcService 就可以注册一个 gRpc 服务
5. `GrpcClientBeanPostProcessor` 这个类处理使用了上述注解的自动注入
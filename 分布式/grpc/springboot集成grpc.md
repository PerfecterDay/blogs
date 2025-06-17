# Springboot 集成 grpc

## grpc-spring-boot-starter 配置
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
5. `GrpcClientBeanPostProcessor` 这个类处理使用了 `@GrpcClient` 注解的自动注入
6. `AnnotationGrpcServiceDiscoverer` 这个类处理了 `@GrpcService` 注解的自动注入


`GrpcServerLifecycle` --> `createAndStartGrpcServer` 等价于我们使用原生 grpc 开发时使用的 ：
```
Server server =
    NettyServerBuilder.forPort(12345)
        .addService(ProtoReflectionService.newInstance()).build();
```


## 配置 ServerInterceptor
向您的服务端添加 ` ServerInterceptor` 的三种方式。

+ 使用 @GrpcGlobalServerIntercetor 注解或者使用 GlobalServerIntercetorConfigurer 将 ServerInterceptor 定义为全局拦截器
+ 在 @GrpcService#interceptors 或 @GrpcService#interceptorNames 字段中明确列出
+ 使用 ` GrpcServerConfigurer 并调用 serverBuilder.intercept(ServerInterceptor interceptor)` 方法

## GrpcServerConfigurer
Grpc 服务端配置器允许您将自定义配置添加到 gRPC 的 ServerBuilder。
```
@Bean
public GrpcServerConfigurer keepAliveServerConfigurer() {
    return serverBuilder -> {
        if (serverBuilder instanceof NettyServerBuilder) {
            ((NettyServerBuilder) serverBuilder)
                    .keepAliveTime(30, TimeUnit.SECONDS)
                    .keepAliveTimeout(5, TimeUnit.SECONDS)
                    .permitKeepAliveWithoutCalls(true);
        }
    };
}
```
# BootStrap 引导 Channel 的过程

```
Bootstrap b = new Bootstrap().group(group)
        .option(ChannelOption.TCP_NODELAY, true)
        .channelFactory(new MyChannelFactoryImpl(group))
        .handler(new ChannelInitializer<SocketChannel>() {
            @Override
            public void initChannel(SocketChannel ch) throws Exception {
                ChannelPipeline p = ch.pipeline();
                if (sslCtx != null) {
                    p.addLast(sslCtx.newHandler(ch.alloc(), HOST, PORT));
                }
                //p.addLast(new LoggingHandler(LogLevel.INFO));
                p.addLast(new EchoClientHandler());
            }
        });

 // Start the client.
ChannelFuture f = b.connect(HOST, PORT).sync();
```

调用栈如下：
<center><img src="pics/bootstrap-connect-1.png" alt=""></center>

`connect` 方法被调用后，最终会调用到 `AbstractBootstrap` 的 `initAndRegister()` 方法:
```
final ChannelFuture initAndRegister() {
    Channel channel = null;
    
    channel = channelFactory.newChannel(); //调用 ChannelFactory 的 newChannel() 获取 Channel 实例
    init(channel); //
    
    ChannelFuture regFuture = config().group().register(channel);
    if (regFuture.cause() != null) {
        if (channel.isRegistered()) {
            channel.close();
        } else {
            channel.unsafe().closeForcibly();
        }
    }
    return regFuture;
}
```

首先会调用 `ChannelFactory` 的 `public Channel newChannel()` 方法获取到一个 channel 实例。  
其次，`init(channel)` 方法根据 Bootstrap 的配置，初始化 channel 实例。
最后，会调用 `EventLoopGroup` 的 `ChannelFuture register(Channel channel)` 将 channel 注册到 `EventLoop` 。在这个过程中会调用注册到 `Bootstrap` 的 `ChannelHandler` 。


#### 客户端发送请求之前，必须初始化连接，连接必须要经过初始化认证
```
package io.netty.channel;

/**
 * Creates a new {@link Channel}.
 */
@SuppressWarnings({ "ClassNameSameAsAncestorName", "deprecation" })
public interface ChannelFactory<T extends Channel> extends io.netty.bootstrap.ChannelFactory<T> {
    /**
     * Creates a new channel.
     */
    @Override
    T newChannel();
}
```

实现上述的 `ChannelFactory` 接口，在实现里边维护若干个初始化过的链接。 每次调用 `newChannel()` 时返回可用的链接。
然后在创建 BoostStrap 时配置 channelFactory：
```
Bootstrap b = new Bootstrap().group(group)
             .option(ChannelOption.TCP_NODELAY, true)
             .channelFactory(new MyChannelFactoryImpl<NioSocketChannel>())
```
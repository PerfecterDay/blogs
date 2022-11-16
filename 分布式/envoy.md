## Envoy
{docsify-updated}


### 线程模型
Envoy 使用单进程-多线程架构。一个 primary 线程处理各种轻量协调任务，同时多个 worker 线程处理监听、过滤、转发。 当一个连接被监听器接受，连接的剩余生命周期将绑定在当前 worker 线程。这使得 Envoy 大部分代码近似单线程运行（高度并行）， 只有少量的复杂代码用于实现 worker 线程之间的协调。Envoy 基本实现了 100% 的非阻塞，对于大多数工作负载， 我们建议将 worker 线程数配置为物理机器的核心数。

### 监听器
每个监听器都独立配置了多个**过滤器链**，其中根据其匹配条件选择某个过滤器链。 一个独立的过滤器链由一个或多个网络层(L3/L4)过滤器组成。 当监听器上接收到新连接时，会选择适当的过滤器链，接着实例化配置的本地筛选器堆栈和处理后续事件。
监听器还可以选择配置一些监听过滤器。 这些过滤器在网络层过滤器之前处理，并且有机会去操作连接元数据，这样通常是为了影响后续过滤器或集群如何处理连接。
还可以通过监听器发现服务 (LDS) 动态获取监听器。

网络层过滤器（filter_chains->filters）
监听过滤器（listener_filters）

### 外部授权
外部授权服务群集可以是静态配置的，也可以是通过 集群服务发现 配置的。如果在请求到达时外部服务不可用，则该请求是否被授权由 网络层过滤器 或 HTTP 过滤器 中的 failure_mode_allow 配置项的设置决定。如果将其设置为 true，则该请求将被放行（故障打开），否则将被拒绝。 默认设置为 false。



```
sudo docker run -idt --name envoy --restart=always -e TZ="Asia/Shanghai" -e loglevel=debug -v /etc/localtime:/etc/localtime -v "/root/dev/envoy/logs:/tmp" -v "/root/dev/envoy/test.cer:/etc/envoy/test.cer" -v "/root/dev/envoy/testserver.key:/etc/envoy/test.key" -v "/root/dev/envoy/envoy.yaml:/etc/envoy/envoy.yaml" --net=host  4b976b6b0e19
```

默认情况下，Docker 镜像将以构建时创建的 envoy 用户来运行。 envoy uid 和 gid 的默认值都是 101 。
envoy 用户的 uid 和 gid 可以在运行时使用 ENVOY_UID 和 ENVOY_GID 这两个环境变量来设定。这也可以在 Docker 命令行中来完成设定
`$ docker run -d --name envoy -e ENVOY_UID=777 -e ENVOY_GID=777 -p 9901:9901 -p 10000:10000 envoy:v1`

如果你把应用程序日志、管理和访问日志输出到一个文件，envoy 用户将需要足够的权限来写这个文件。这个可以通过设置 ENVOY_UID 或者通过将文件变成 envoy 用户可写的方法来实现。
```
mkdir logs
chown 777 logs
docker run -d -v `pwd`/logs:/var/log --name envoy -e ENVOY_UID=777 -p 9901:9901 -p 10000:10000 envoy:v1
```
随后，你可以配置 envoy 将日志文件输出在 /var/log 文件里。

有一种可以不用改变任何文件的权限或者在容器内部使用 root 用户的方法就是在启动容器的时候使用宿主机用户的 uid :
`docker run -d --name envoy -e ENVOY_UID=`id -u` -p 9901:9901 -p 10000:10000 envoy:v1`



`-e loglevel=debug`: 设置日志级别

logs 文件夹报 permission denied 错误 -> 必须进入容器内部后使用 `chown envoy /tmp` 改变目录所有者，然后运行容器
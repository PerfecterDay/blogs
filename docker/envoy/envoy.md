## Envoy
{docsify-updated}
> https://github.com/envoyproxy/envoy


<center><img src="pics/envoy-architecture.webp" width="60%"></center>

### 配置输出日志
envoy -c envoy-demo.yaml --log-path logs/custom.log


### 线程模型
Envoy 使用单进程-多线程架构。一个 primary 线程处理各种轻量协调任务，同时多个 worker 线程处理监听、过滤、转发。 当一个连接被监听器接受，连接的剩余生命周期将绑定在当前 worker 线程。这使得 Envoy 大部分代码近似单线程运行（高度并行）， 只有少量的复杂代码用于实现 worker 线程之间的协调。Envoy 基本实现了 100% 的非阻塞，对于大多数工作负载， 我们建议将 worker 线程数配置为物理机器的核心数。

### 监听器
每个监听器都独立配置了多个**过滤器链**，其中根据其匹配条件选择某个过滤器链。 一个独立的过滤器链由一个或多个网络层(L3/L4)过滤器组成。 当监听器上接收到新连接时，会选择适当的过滤器链，接着实例化配置的本地筛选器堆栈和处理后续事件。
监听器还可以选择配置一些监听过滤器。 这些过滤器在网络层过滤器之前处理，并且有机会去操作连接元数据，这样通常是为了影响后续过滤器或集群如何处理连接。
还可以通过监听器发现服务 (LDS) 动态获取监听器。

+ 网络层过滤器（filter_chains->filters）
+ 监听过滤器（listener_filters）

### 外部授权
外部授权服务群集可以是静态配置的，也可以是通过 集群服务发现 配置的。如果在请求到达时外部服务不可用，则该请求是否被授权由 网络层过滤器 或 HTTP 过滤器 中的 failure_mode_allow 配置项的设置决定。如果将其设置为 true，则该请求将被放行（故障打开），否则将被拒绝。 默认设置为 false。


### 路由匹配
当 Envoy 匹配到一条路由时，它使用如下流程：
1. HTTP 请求的 host 或 :authority 头部会和一个虚拟主机相匹配。
2. 虚拟主机中的每一个路由条目都会按顺序地逐个被检查。 如果匹配到了，则使用此路由且不再做其它路由检查。
3. 虚拟主机中的每一个虚拟集群都会独立地按顺序地被逐个检查。如果匹配到了，则使用此虚拟集群且不再做其它虚拟集群检查。

`typed_per_filter_config` 字段可以用来提供过滤器的特定路由配置。键应该与过滤器的名称相匹配，比如 `envoy.filters.http.buffer` 是指HTTP缓冲区过滤器。这个字段的使用是针对过滤器的；关于是否使用以及如何使用，请看HTTP过滤器的文档。


### Access Log

1. `%RESPONSE_FLAGS%`
如果有的话，表示响应或者连接的附加详情。 对于 TCP 连接，说明中提到的响应码不适用。 可能的值如下：

HTTP 和 TCP
+ UH: 附加在 503 响应状态码，表示上游集群中无健康的上游主机。
+ UF: 附加在 503 响应状态码，表示上游连接失败
+ UO: 附加在 503 响应状态码，表示上游溢出 (熔断) 。
+ NR: 附加在 404 响应状态码，表示无给定请求的 路由配置 ，或者对于一个下游连接没有匹配的过滤器链。
+ URX: 请求因为达到了上游重试限制 (HTTP) 或者最大连接尝试 (TCP) 而被拒绝。

HTTP 独有
+ DC: 下游连接中断。
+ LH: 附加在 503 响应状态码，本地服务 健康检查请求 失败。
+ UT: 附加在 504 响应状态码，上游请求超时。
+ LR: 附加在 503 响应状态码，连接在本地被重置。
+ UR: 附加在 503 响应状态码，连接在远程被重置。
+ UC: 附加在 503 响应状态码，上游连接中断。
+ DI: 通过 故障注入 使请求处理被延迟一个指定的时间。
+ FI: 通过 故障注入 使请求被终止掉并带有一个相应码。
+ RL: 附加在 429 相应状态码，请求被 HTTP 限流过滤器 本地限流。
+ UAEX: 请求被外部授权服务拒绝。
+ RLSE: 因限流服务出现错误而拒绝请求。
+ IH: 附加在 400 响应状态码，请求被拒绝因为他为 strictly-checked header 设置了一个无效值。
+ SI: 附加在 408 相应状态码，流闲置超时。
+ DPE: 下游请求存在一个 HTTP 协议错误。
+ UMSDR: 上游请求达到最大流持续时长


### 配置websocket
 ```
 - match:
        safe_regex: { google_re2: {}, regex: "^/yichat.*" }
    typed_per_filter_config:
        envoy.filters.http.ext_authz:
        "@type": type.googleapis.com/envoy.extensions.filters.http.ext_authz.v3.ExtAuthzPerRoute
        disabled: true
    route: { cluster: kefu-cluster-1, timeout: { seconds: 0 }, upgrade_configs: [{upgrade_type: websocket}] }
 ```

1. uatfile upload host 不匹配返回403 : auto_host_rewrite
   `route: { cluster: uatfile-cluster, timeout: { seconds: 10 } ,auto_host_rewrite: true}`
2. 设置超时时间: timeout
   `route: { cluster: uatfile-cluster, timeout: { seconds: 60 } ,auto_host_rewrite: true}`
3. 上游服务配置 https : transport_socket
   ```
   clusters:
     - name: service_envoyproxy_io
       type: LOGICAL_DNS
       # Comment out the following line to test on v6 networks
       dns_lookup_family: V4_ONLY
       load_assignment:
         cluster_name: service_envoyproxy_io
         endpoints:
         - lb_endpoints:
           - endpoint:
               address:
                 socket_address:
                   address: www.envoyproxy.io
                   port_value: 443
       transport_socket:
         name: envoy.transport_sockets.tls
         typed_config:
           "@type": type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.UpstreamTlsContext
   ```
4. uatfile 返回413,Payload Too Large: per_connection_buffer_limit_bytes
   ```
   clusters:
    name: cluster_0
    connect_timeout: 5s
    per_connection_buffer_limit_bytes: 16000000
    load_assignment:
      cluster_name: some_service
      endpoints:
        - lb_endpoints:
          - endpoint:
              address:
                socket_address:
                  address: ::1
                  port_value: 46685
   ```

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

(access_log)[https://www.envoyproxy.io/docs/envoy/latest/configuration/observability/access_log/usage#config-access-log]


### 重写请求路径，禁用三防
```
 - match:
        prefix: "/h5/fund"
    typed_per_filter_config:
        envoy.filters.http.ext_authz:
        "@type": type.googleapis.com/envoy.extensions.filters.http.ext_authz.v3.ExtAuthzPerRoute
        disabled: true
    route: { cluster: financial-management-cluster, timeout: { seconds: 10 } , regex_rewrite: {pattern: {regex: "/h5/fund", google_re2: {}},substitution: "/fund"} }
    - match:
        safe_regex: { google_re2: {}, regex: "^/[service|yichat].*" }
    typed_per_filter_config:
        envoy.filters.http.ext_authz:
        "@type": type.googleapis.com/envoy.extensions.filters.http.ext_authz.v3.ExtAuthzPerRoute
        disabled: true
    route: { cluster: kefu-cluster, timeout: { seconds: 10 } }
```
# envoy 配置
{docsify-updated}

> 配置参考：https://www.envoyproxy.io/docs/envoy/latest/configuration/configuration

### 配置生成器
Envoy 配置会变得相对复杂。在 Lyft 我们使用 jinja 模板使配置变得更加易于创建和管理。发行版源代码包含一个配置生成器的版本，它与我们在 Lyft 中使用的配置生成器近似一致。我们也为上述的三个场景中的每一个都包含了示例配置模板。

+ 生成器脚本： configs/configgen.py
+ 服务对服务模板： configs/envoy_service_to_service_v2.template.yaml
+ 前置代理模板： configs/envoy_front_proxy_v2.template.yaml
+ 双重代理模板： configs/envoy_double_proxy_v2.template.yaml

从仓库的根目录运行以下命令生成示例配置：
```
mkdir -p generated/configs
bazel build //configs:example_configs
tar xvf $PWD/bazel-out/k8-fastbuild/bin/configs/example_configs.tar -C generated/configs
```

### Access Log
> https://cloudnative.to/envoy/configuration/observability/access_log/usage.html#config-access-log-format-dictionaries

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

### 配置输出日志
envoy -c envoy-demo.yaml --log-path logs/custom.log

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

### 路由的配置
重写请求路径/对特定路由禁用三防插件/转发时增加请求头
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
    - match:
        safe_regex: { google_re2: {}, regex: "^/gtjaiback.*" }
      route: { cluster: bpm-cluster, timeout: { seconds: 10 } }
      request_headers_to_add: 
      - header:
          key: "appId"
          value: "1e6c3a95-552a-5d04-38ad-2f4e238236b6"
```

### 上有集群断路器配置

断路是分布式系统的重要组成部分。在分布式系统中最好是迅速失败，并尽快向下游施加反压。 Envoy 网络的主要好处之一是，Envoy 在网络级别强制执行断路限制，而不是必须独立给每个应用程序配置和编码。Envoy 支持各种类型的全分布式（非协调）断路：

**集群最大连接数** ：Envoy 和上游集群中所有主机建立的最大连接数。如果该断路器溢出，集群的 upstream_cx_overflow 计数器将增加。所有的连接，不管是活动的还是空闲的，都会计入这个计数器并由它来限制。即使这个断路器已经溢出，Envoy 也会确保集群负载均衡选择的主机至少有一个连接分配。这就意味着：集群的 upstream_cx_active 计数可能会高于集群最大连接断路器，其上限为`集群最大连接数 +（集群的端点数）*（集群的连接池）`。这个边界适用于所有工作者线程的连接数之和。参见 连接池，了解一个集群可能有多少个连接池。

**集群最大待处理请求** ：在等待就绪连接池连接时排队的最大请求数。每当没有足够的上游连接可用来立即调度请求时，请求就会被添加到待处理请求列表中。对于 HTTP/2 连接，如果 最大并发流 和 每个连接的最大请求数 没有配置，所有的请求都会在同一个连接上被复用，所以只有在还没有建立连接的时候，才会触发这个断路器。如果这个断路器溢出，集群的 `upstream_rq_pending_overflow` 计数器将递增。

**集群最大请求量** ：在任何特定时间对集群中所有主机的最大请求数。如果该断路器溢出，集群的 `upstream_rq_pending_overflow` 计数器将增加。

**集群最大有效重试** ：集群中ß所有主机在任何特定时间内都可以进行的最大重试次数。一般来说，我们建议使用 重试预算；但是，如果静态断路是首选项，则应该积极断路重试。这样可以允许零星故障的重试，但总体重试量不能爆炸，不能造成大规模的级联故障。如果这个断路器溢出，集群的 `upstream_rq_retry_overflow` 计数器会递增。

**集群最大并发连接池** ：可并发实例化的最大连接池数量。有些功能，如 源监听器过滤器，可以创建无限制数量的连接池。当一个集群用尽了它的并发连接池，它将尝试回收一个空闲的连接池。如果不能，那么断路器将溢出。这与 集群最大连接数 不同的是，连接池永远不会超时，而连接通常会超时。连接会自动清理，而连接池不会。需要注意的是，为了让连接池发挥作用，它至少需要一个上游连接，所以这个值很可能不应该大于 集群最大连接数。如果这个断路器溢出，集群的 `upstream_cx_pool_overflow` 计数器将递增。

每个断路器的限制是可配置的，并按每个上游集群和每个优先级进行跟踪。这使得分布式系统的不同组件可以独立调整，并有不同的限制。可以通过统计信息来观察这些断路器的实时状态，包括在断路器打开之前剩余的资源数量。

工作线程共享断路器限制，即如果活动连接阈值为 500，工作线程 1 有 498 个活动连接，那么工作线程 2 只能再分配 2 个连接。由于实现最终是一致的，线程之间的竞赛可能会让限制可能超出。

断路器是默认启用的，并且有适度的默认值，例如每个集群有 1024 个连接。要禁用断路器，请将其阈值设置为允许的最高值。

需要注意的是，在 HTTP 请求中，断路会导致 `x-envoy-overloaded` 头被路由器过滤器设置。
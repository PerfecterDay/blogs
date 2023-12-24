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


查看 envoy 的状态统计信息：
https://www.envoyproxy.io/docs/envoy/latest/configuration/upstream/cluster_manager/cluster_stats.html

https://www.envoyproxy.io/docs/envoy/latest/operations/stats_overview

upstream_rq_pending_total

cluster.user-cluster.bind_errors: 0
cluster.user-cluster.circuit_breakers.default.cx_open: 0
cluster.user-cluster.circuit_breakers.default.cx_pool_open: 0
cluster.user-cluster.circuit_breakers.default.rq_open: 0
cluster.user-cluster.circuit_breakers.default.rq_pending_open: 0
cluster.user-cluster.circuit_breakers.default.rq_retry_open: 0
cluster.user-cluster.circuit_breakers.high.cx_open: 0
cluster.user-cluster.circuit_breakers.high.cx_pool_open: 0
cluster.user-cluster.circuit_breakers.high.rq_open: 0
cluster.user-cluster.circuit_breakers.high.rq_pending_open: 0
cluster.user-cluster.circuit_breakers.high.rq_retry_open: 0
cluster.user-cluster.default.total_match_count: 37270
cluster.user-cluster.external.upstream_rq_200: 5014
cluster.user-cluster.external.upstream_rq_2xx: 5014
cluster.user-cluster.external.upstream_rq_completed: 5014
cluster.user-cluster.health_check.attempt: 53192
cluster.user-cluster.health_check.degraded: 0
cluster.user-cluster.health_check.failure: 0
cluster.user-cluster.health_check.healthy: 1
cluster.user-cluster.health_check.network_failure: 0
cluster.user-cluster.health_check.passive_failure: 0
cluster.user-cluster.health_check.success: 53192
cluster.user-cluster.health_check.verify_cluster: 0
cluster.user-cluster.http1.dropped_headers_with_underscores: 0
cluster.user-cluster.http1.metadata_not_supported_error: 0
cluster.user-cluster.http1.requests_rejected_with_underscores_in_headers: 0
cluster.user-cluster.http1.response_flood: 0
cluster.user-cluster.lb_healthy_panic: 0
cluster.user-cluster.lb_local_cluster_not_ok: 0
cluster.user-cluster.lb_recalculate_zone_structures: 0
cluster.user-cluster.lb_subsets_active: 0
cluster.user-cluster.lb_subsets_created: 0
cluster.user-cluster.lb_subsets_fallback: 0
cluster.user-cluster.lb_subsets_fallback_panic: 0
cluster.user-cluster.lb_subsets_removed: 0
cluster.user-cluster.lb_subsets_selected: 0
cluster.user-cluster.lb_zone_cluster_too_small: 0
cluster.user-cluster.lb_zone_no_capacity_left: 0
cluster.user-cluster.lb_zone_number_differs: 0
cluster.user-cluster.lb_zone_routing_all_directly: 0
cluster.user-cluster.lb_zone_routing_cross_zone: 0
cluster.user-cluster.lb_zone_routing_sampled: 0
cluster.user-cluster.max_host_weight: 1
cluster.user-cluster.membership_change: 1
cluster.user-cluster.membership_degraded: 0
cluster.user-cluster.membership_excluded: 0
cluster.user-cluster.membership_healthy: 1
cluster.user-cluster.membership_total: 1
cluster.user-cluster.original_dst_host_invalid: 0
cluster.user-cluster.retry_or_shadow_abandoned: 0
cluster.user-cluster.update_attempt: 37270
cluster.user-cluster.update_empty: 0
cluster.user-cluster.update_success: 37270
cluster.user-cluster.upstream_cx_active: 2
cluster.user-cluster.upstream_cx_close_notify: 0
cluster.user-cluster.upstream_cx_connect_attempts_exceeded: 0
cluster.user-cluster.upstream_cx_connect_fail: 0
cluster.user-cluster.upstream_cx_connect_timeout: 0
cluster.user-cluster.upstream_cx_destroy: 58
cluster.user-cluster.upstream_cx_destroy_local: 58
cluster.user-cluster.upstream_cx_destroy_local_with_active_rq: 0
cluster.user-cluster.upstream_cx_destroy_remote: 0
cluster.user-cluster.upstream_cx_destroy_remote_with_active_rq: 0
cluster.user-cluster.upstream_cx_destroy_with_active_rq: 0
cluster.user-cluster.upstream_cx_http1_total: 60
cluster.user-cluster.upstream_cx_http2_total: 0
cluster.user-cluster.upstream_cx_http3_total: 0
cluster.user-cluster.upstream_cx_idle_timeout: 58
cluster.user-cluster.upstream_cx_max_requests: 0
cluster.user-cluster.upstream_cx_none_healthy: 0
cluster.user-cluster.upstream_cx_overflow: 0
cluster.user-cluster.upstream_cx_pool_overflow: 0
cluster.user-cluster.upstream_cx_protocol_error: 0
cluster.user-cluster.upstream_cx_rx_bytes_buffered: 414
cluster.user-cluster.upstream_cx_rx_bytes_total: 9591488
cluster.user-cluster.upstream_cx_total: 60
cluster.user-cluster.upstream_cx_tx_bytes_buffered: 0
cluster.user-cluster.upstream_cx_tx_bytes_total: 3906872
cluster.user-cluster.upstream_flow_control_backed_up_total: 0
cluster.user-cluster.upstream_flow_control_drained_total: 0
cluster.user-cluster.upstream_flow_control_paused_reading_total: 0
cluster.user-cluster.upstream_flow_control_resumed_reading_total: 0
cluster.user-cluster.upstream_internal_redirect_failed_total: 0
cluster.user-cluster.upstream_internal_redirect_succeeded_total: 0
cluster.user-cluster.upstream_rq_200: 5014
cluster.user-cluster.upstream_rq_2xx: 5014
cluster.user-cluster.upstream_rq_active: 0
cluster.user-cluster.upstream_rq_cancelled: 0
cluster.user-cluster.upstream_rq_completed: 5014
cluster.user-cluster.upstream_rq_maintenance_mode: 0
cluster.user-cluster.upstream_rq_max_duration_reached: 0
cluster.user-cluster.upstream_rq_pending_active: 0
cluster.user-cluster.upstream_rq_pending_failure_eject: 0
cluster.user-cluster.upstream_rq_pending_overflow: 0
cluster.user-cluster.upstream_rq_pending_total: 60
cluster.user-cluster.upstream_rq_per_try_timeout: 0
cluster.user-cluster.upstream_rq_retry: 0
cluster.user-cluster.upstream_rq_retry_backoff_exponential: 0
cluster.user-cluster.upstream_rq_retry_backoff_ratelimited: 0
cluster.user-cluster.upstream_rq_retry_limit_exceeded: 0
cluster.user-cluster.upstream_rq_retry_overflow: 0
cluster.user-cluster.upstream_rq_retry_success: 0
cluster.user-cluster.upstream_rq_rx_reset: 0
cluster.user-cluster.upstream_rq_timeout: 0
cluster.user-cluster.upstream_rq_total: 5014
cluster.user-cluster.upstream_rq_tx_reset: 0
cluster.user-cluster.version: 0
cluster.user-cluster.external.upstream_rq_time: P0(nan,5.0) P25(nan,23.894715111478117) P50(nan,25.24342105263158) P75(nan,33.4685534591195) P90(nan,68.98000000000002) P95(nan,134.92592592592598) P99(nan,379.5333333333322) P99.5(nan,559.3000000000029) P99.9(nan,1299.5333333333292) P100(nan,3000.0)
cluster.user-cluster.upstream_cx_connect_ms: P0(nan,0.0) P25(nan,0.0) P50(nan,0.0) P75(nan,0.0) P90(nan,0.0) P95(nan,0.0) P99(nan,1.0399999999999998) P99.5(nan,1.0700000000000003) P99.9(nan,1.0939999999999999) P100(nan,1.1)
cluster.user-cluster.upstream_cx_length_ms: P0(nan,3600000.0) P25(nan,3669047.619047619) P50(nan,5200000.0) P75(nan,10500000.0) P90(nan,14200000.000000004) P95(nan,16099999.999999994) P99(nan,20420000.0) P99.5(nan,20710000.0) P99.9(nan,20942000.0) P100(nan,21000000.0)
cluster.user-cluster.upstream_rq_time: P0(nan,5.0) P25(nan,23.894715111478117) P50(nan,25.24342105263158) P75(nan,33.4685534591195) P90(nan,68.98000000000002) P95(nan,134.92592592592598) P99(nan,379.5333333333322) P99.5(nan,559.3000000000029) P99.9(nan,1299.5333333333292) P100(nan,3000.0)



cluster.trade-cluster.assignment_stale: 0
cluster.trade-cluster.assignment_timeout_received: 0
cluster.trade-cluster.bind_errors: 0
cluster.trade-cluster.circuit_breakers.default.cx_open: 0
cluster.trade-cluster.circuit_breakers.default.cx_pool_open: 0
cluster.trade-cluster.circuit_breakers.default.rq_open: 0
cluster.trade-cluster.circuit_breakers.default.rq_pending_open: 0
cluster.trade-cluster.circuit_breakers.default.rq_retry_open: 0
cluster.trade-cluster.circuit_breakers.high.cx_open: 0
cluster.trade-cluster.circuit_breakers.high.cx_pool_open: 0
cluster.trade-cluster.circuit_breakers.high.rq_open: 0
cluster.trade-cluster.circuit_breakers.high.rq_pending_open: 0
cluster.trade-cluster.circuit_breakers.high.rq_retry_open: 0
cluster.trade-cluster.default.total_match_count: 37196
cluster.trade-cluster.external.upstream_rq_200: 6848
cluster.trade-cluster.external.upstream_rq_2xx: 6848
cluster.trade-cluster.external.upstream_rq_completed: 6848
cluster.trade-cluster.health_check.attempt: 53113
cluster.trade-cluster.health_check.degraded: 0
cluster.trade-cluster.health_check.failure: 0
cluster.trade-cluster.health_check.healthy: 1
cluster.trade-cluster.health_check.network_failure: 0
cluster.trade-cluster.health_check.passive_failure: 0
cluster.trade-cluster.health_check.success: 53113
cluster.trade-cluster.health_check.verify_cluster: 0
cluster.trade-cluster.http1.dropped_headers_with_underscores: 0
cluster.trade-cluster.http1.metadata_not_supported_error: 0
cluster.trade-cluster.http1.requests_rejected_with_underscores_in_headers: 0
cluster.trade-cluster.http1.response_flood: 0
cluster.trade-cluster.lb_healthy_panic: 0
cluster.trade-cluster.lb_local_cluster_not_ok: 0
cluster.trade-cluster.lb_recalculate_zone_structures: 0
cluster.trade-cluster.lb_subsets_active: 0
cluster.trade-cluster.lb_subsets_created: 0
cluster.trade-cluster.lb_subsets_fallback: 0
cluster.trade-cluster.lb_subsets_fallback_panic: 0
cluster.trade-cluster.lb_subsets_removed: 0
cluster.trade-cluster.lb_subsets_selected: 0
cluster.trade-cluster.lb_zone_cluster_too_small: 0
cluster.trade-cluster.lb_zone_no_capacity_left: 0
cluster.trade-cluster.lb_zone_number_differs: 0
cluster.trade-cluster.lb_zone_routing_all_directly: 0
cluster.trade-cluster.lb_zone_routing_cross_zone: 0
cluster.trade-cluster.lb_zone_routing_sampled: 0
cluster.trade-cluster.max_host_weight: 1
cluster.trade-cluster.membership_change: 1
cluster.trade-cluster.membership_degraded: 0
cluster.trade-cluster.membership_excluded: 0
cluster.trade-cluster.membership_healthy: 1
cluster.trade-cluster.membership_total: 1
cluster.trade-cluster.original_dst_host_invalid: 0
cluster.trade-cluster.retry_or_shadow_abandoned: 0
cluster.trade-cluster.update_success: 37196
cluster.trade-cluster.upstream_cx_active: 0
cluster.trade-cluster.upstream_cx_close_notify: 0
cluster.trade-cluster.upstream_cx_connect_attempts_exceeded: 0
cluster.trade-cluster.upstream_cx_connect_fail: 0
cluster.trade-cluster.upstream_cx_connect_timeout: 0
cluster.trade-cluster.upstream_cx_destroy: 510
cluster.trade-cluster.upstream_cx_destroy_local: 510
cluster.trade-cluster.upstream_cx_destroy_local_with_active_rq: 13
cluster.trade-cluster.upstream_cx_destroy_remote: 0
cluster.trade-cluster.upstream_cx_destroy_remote_with_active_rq: 0
cluster.trade-cluster.upstream_cx_destroy_with_active_rq: 13
cluster.trade-cluster.upstream_cx_http1_total: 510
cluster.trade-cluster.upstream_cx_http2_total: 0
cluster.trade-cluster.upstream_cx_http3_total: 0
cluster.trade-cluster.upstream_cx_idle_timeout: 497
cluster.trade-cluster.upstream_cx_max_requests: 0
cluster.trade-cluster.upstream_cx_none_healthy: 0
cluster.trade-cluster.upstream_cx_overflow: 0
cluster.trade-cluster.upstream_cx_pool_overflow: 0
cluster.trade-cluster.upstream_cx_protocol_error: 0
cluster.trade-cluster.upstream_cx_rx_bytes_buffered: 0
cluster.trade-cluster.upstream_cx_rx_bytes_total: 12011905
cluster.trade-cluster.upstream_cx_total: 510
cluster.trade-cluster.upstream_cx_tx_bytes_buffered: 0
cluster.trade-cluster.upstream_cx_tx_bytes_total: 5225667
cluster.trade-cluster.upstream_flow_control_backed_up_total: 0
cluster.trade-cluster.upstream_flow_control_drained_total: 0
cluster.trade-cluster.upstream_flow_control_paused_reading_total: 0
cluster.trade-cluster.upstream_flow_control_resumed_reading_total: 0
cluster.trade-cluster.upstream_internal_redirect_failed_total: 0
cluster.trade-cluster.upstream_internal_redirect_succeeded_total: 0
cluster.trade-cluster.upstream_rq_200: 6848
cluster.trade-cluster.upstream_rq_2xx: 6848
cluster.trade-cluster.upstream_rq_active: 0
cluster.trade-cluster.upstream_rq_cancelled: 0
cluster.trade-cluster.upstream_rq_completed: 6848
cluster.trade-cluster.upstream_rq_maintenance_mode: 0
cluster.trade-cluster.upstream_rq_max_duration_reached: 0
cluster.trade-cluster.upstream_rq_pending_active: 0
cluster.trade-cluster.upstream_rq_pending_failure_eject: 0
cluster.trade-cluster.upstream_rq_pending_overflow: 0
cluster.trade-cluster.upstream_rq_pending_total: 510
cluster.trade-cluster.upstream_rq_per_try_timeout: 0
cluster.trade-cluster.upstream_rq_retry: 0
cluster.trade-cluster.upstream_rq_retry_backoff_exponential: 0
cluster.trade-cluster.upstream_rq_retry_backoff_ratelimited: 0
cluster.trade-cluster.upstream_rq_retry_limit_exceeded: 0
cluster.trade-cluster.upstream_rq_retry_overflow: 0
cluster.trade-cluster.upstream_rq_retry_success: 0
cluster.trade-cluster.upstream_rq_rx_reset: 0
cluster.trade-cluster.upstream_rq_timeout: 0


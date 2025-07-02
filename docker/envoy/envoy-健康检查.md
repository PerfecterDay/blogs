# Envoy 健康检查
{docsify-updated}

> https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/upstream/health_checking#arch-overview-health-checking

主动健康检查可根据每个上游群集进行配置。主动健康检查和 EDS 服务发现类型是相辅相成的。不过，在其他情况下，即使使用其他服务发现类型，也需要进行主动健康检查。Envoy 支持三种不同类型的健康检查和各种设置（检查时间间隔、标记主机不健康前所需的失败次数、标记主机健康前所需的成功次数等）：
1. HTTP：在 HTTP 健康检查期间，Envoy 会向上游主机发送 HTTP 请求。默认情况下，如果主机是健康的，它就会得到 200 的响应。预期响应代码和重试响应代码是可配置的。如果上游主机希望立即通知下游主机不再向其转发流量，它可以返回非预期或非可恢复状态代码（默认为任何非 200 代码）。
2. gRPC： 在 gRPC 健康检查期间，Envoy 会向上游主机发送 gRPC 请求。默认情况下，如果主机是健康的，它希望得到 200 的响应。
3. L3/L4：在 L3/L4 健康检查期间，Envoy 会向上游主机发送一个可配置的字节缓冲区。如果主机被认为是健康的，它就会在响应中回传该字节缓冲区。Envoy 还支持仅连接的 L3/L4 健康检查。
4. Redis： Envoy 会发送一条 Redis PING 命令，并期待得到 PONG 响应。上游 Redis 服务器可能会以 PONG 以外的任何其他方式回应，从而导致主动健康检查立即失败。作为选项，Envoy 可以对用户指定的密钥执行 EXISTS。如果键不存在，则视为健康检查通过。这样，用户就可以将指定的键值设置为任意值，然后等待流量耗尽，从而标记 Redis 实例进行维护。
5. Thrift： Envoy 会发送 Thrift 请求，并期待成功响应。上游主机也可能响应异常，导致健康检查失败。

Envoy 健康检查的默认行为：
+ 当该实例健康检查失败次数达到 `unhealthy_threshold`，Envoy 会将其标记为 unhealthy；
+ Envoy 会将其从负载均衡池中移除；
+ 不再向其转发流量。但是在某些特殊情况下（比如集群仅 1 个节点时），envoy 仍然会转发流量给不健康的节点。

默认情况下，Envoy 会启用 “Healthy Panic Mode”，也就是：如果健康主机比例低于 `healthy_panic_threshold` （默认是 50%），**Envoy 会忽略健康检查结果，继续向所有主机转发流量，避免“完全不可用”**。
所以：
+ 如果只有 1 台主机且它 unhealthy ⇒ 健康比例 0%；
+ 小于默认的 50% ⇒ panic 模式触发 ⇒ 仍然会继续向该主机转发流量！

也就是只有健康比例大于等于这个阈值配置时，才会停止流量转发。所以，如果想要即使只有一个节点当发现不健康时也不转发流量，就需要讲这个阈值设置为0：
```
common_lb_config:
      healthy_panic_threshold:
        value: 0
```

## 健康检查的配置
Envoy 默认不会自动开启健康检查，必须显式在 cluster 配置中添加 `health_checks` 字段。，Envoy 才会对上游服务进行健康探测。示例：
```
clusters:
- name: my_service
  connect_timeout: 0.25s
  type: STRICT_DNS
  lb_policy: ROUND_ROBIN
  load_assignment:
      cluster_name: search-cluster
      endpoints:
      - lb_endpoints:
        - endpoint:
            health_check_config:
              port_value: 8091
              address:
                socket_address:
                  address: search-service-svc
                  port_value: 8091
            address:
              socket_address:
                address: search-service-svc
                port_value: 8000
                ipv4_compat: true
  common_lb_config:
      healthy_panic_threshold:
        value: 0
  health_checks:
  - timeout: 1s
    interval: 5s
    unhealthy_threshold: 3
    healthy_threshold: 2
    http_health_check:
      path: /health
      codec_client_type: HTTP1
```


| 字段                   | 说明                          |
|------------------------|-------------------------------|
| `timeout`              | 单次健康检查超时时间          |
| `interval`             | 健康检查间隔                  |
| `unhealthy_threshold`  | 连续失败多少次视为不健康      |
| `healthy_threshold`    | 连续成功多少次视为恢复健康    |
| `http_health_check.path` | 发起 HTTP 请求的路径       |
| `tcp_health_check`     | TCP 层的健康检查方式（如不用 HTTP） |
| `grpc_health_check`    | GRPC 方式健康检查             |



## 健康检查的日志
默认日志行为：
1. 健康检查成功：
    + 不会每次都记录日志；
    + 只在主机从“不健康”恢复为“健康”状态时（即达到 healthy_threshold）打印一条恢复日志。
2. 健康检查失败：
    + 只在连续失败达到 `unhealthy_threshold` ，主机状态从“健康”变为“不健康”时，打印一条日志；
    + 除非开启了 `always_log_health_check_failures` ，否则中间的失败也不会被记录。

中间的失败/成功都不会单独打印日志。

```
{"health_checker_type":"HTTP","host":{"socket_address":{"protocol":"TCP","address":"192.168.168.129","resolver_name":"","ipv4_compat":false,"port_value":8000}},"cluster_name":"search-cluster","add_healthy_event":{"first_check":true},"timestamp":"2025-07-02T07:55:38.260Z"} --- 健康检查成功的日志

{"health_checker_type":"HTTP","host":{"socket_address":{"protocol":"TCP","address":"192.168.168.129","resolver_name":"","ipv4_compat":false,"port_value":8000}},"cluster_name":"search-cluster","health_check_failure_event":{"failure_type":"ACTIVE","first_check":true},"timestamp":"2025-07-02T06:52:45.039Z"} --- 健康检查失败的日志

{"health_checker_type":"HTTP","host":{"socket_address":{"protocol":"TCP","address":"192.168.168.129","resolver_name":"","ipv4_compat":false,"port_value":8000}},"cluster_name":"search-cluster","eject_unhealthy_event":{"failure_type":"NETWORK"},"timestamp":"2025-07-02T07:58:09.811Z"} --- 健康检查失败的日志
```


## 被动健康检查
# envoy 熔断
{docsify-updated}


Circuit Breaker 触发后，Envoy 会立即返回 本地响应（通常是 503），不再尝试连接上游主机。


## 自定义熔断返回
```
http_filters:
- name: envoy.filters.network.http_connection_manager
  typed_config:
    "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
    stat_prefix: ingress_http
    route_config: {...}
    http_filters:
    - name: envoy.filters.http.router
    local_reply_config:
      mappers:
        - filter:
            response_flag_filter:
              flags:
                - UF  # Upstream Overflow = 熔断触发（请求数满）
          status_code: 529   # 自定义 HTTP 状态码（例如非标准码）
          body_format_override:
            text_format: '{"code":529, "message":"熔断器已触发，服务繁忙"}'
```

`response_flag_filter.flags`:
+ UF（Upstream Overflow）：表示熔断器触发，Envoy 拒绝请求。
+ NR：No Route
+ UH：No healthy upstream
+ UT：Upstream timeout

`status_code`: 设置为你想要返回的 HTTP 状态码，如：
+ 标准码如 429（Too Many Requests）
+ 自定义码如 529（注意客户端可能不识别）

`body_format_override` : 自定义响应体，可以返回 JSON、文本等。
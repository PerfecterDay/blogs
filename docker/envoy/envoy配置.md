# envoy 配置
{docsisy-updated}

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

### 问题列表
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
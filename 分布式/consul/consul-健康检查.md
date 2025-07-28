# consul 健康检查机制
{docsify-updated}

> https://developer.hashicorp.com/consul/docs/register/health-check/vm


## HTTP
```
spring:
  cloud:
    consul:
      discovery:
        health-check-path: /actuator/health
        health-check-interval: 10s
        health-check-critical-timeout: 1m
```
1. 服务宕机，Consul 每 10 秒访问 /actuator/health 失败；
2. Consul 将该实例标记为 critical；
3. 如果服务 在 1 分钟内恢复，Consul 检查再次成功，会将状态从 critical 恢复为 passing；
4. 服务重新可用，对外暴露；
5. 如果服务 宕机时间超过 1 分钟（即超过 deregister_critical_service_after），Consul 会将服务从注册中心彻底移除；
6. 此时即使服务恢复，Consul 也不会再自动“重新注册”，因为它已经忘记该服务了；
7. 除非你的应用在恢复时执行了重新注册逻辑（如 Spring Boot 重新启动后自动注册）。

## TTL 
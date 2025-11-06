# consul 作为配置中心
{docsify-updated}

## consul 中添加配置并查询
UI创建配置值：
<center><img src="pics/config-key.png" alt=""></center>

读取配置值：
```
consul kv get /redis/conf/username
consul kv get /test
```

也可以使用命令行创建配置值：
```
consul kv put redis/conf/pwd test
```
但是注意，此时 key 名不能以 `/` 开头，否则会报错。

## 

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-config</artifactId>
    <version>3.1.2</version>
</dependency>


spring:
  config:
    import: consul
  application:
    name: user-center-service
  cloud:
    consul:
      host: localhost
      port: 8500
      config:
        enabled: true
        format: yaml
        prefix: user-center
        default-context: default
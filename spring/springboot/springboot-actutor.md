## Springboot Actuator
{docsify-updated}

> [docs](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)

Springboot Actuator 可以暴露一些 http 端点让用户能查看 springboot 应用的运行信息。默认的访问 url 是 /actuator/{endpoint} 。从 springboot 2.x 版本开始，默认只暴露 /health 端点。

添加下面的依赖就能开启 Springboot Actuator：
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

如果需要开启其它端点，需要增加下面配置：
```
management:
  endpoints:
    web:
      exposure:
        include: "*"
        exclude: "env"
```

可以通过 url : http://localhost:8050/actuator/ 获取到所有可访问的端点，示例如下：
```
{
	"_links": {
		"self": {
			"href": "http://localhost:8050/actuator",
			"templated": false
		},
		"consul": {
			"href": "http://localhost:8050/actuator/consul",
			"templated": false
		},
		"features": {
			"href": "http://localhost:8050/actuator/features",
			"templated": false
		},
		"caches": {
			"href": "http://localhost:8050/actuator/caches",
			"templated": false
		},
		"caches-cache": {
			"href": "http://localhost:8050/actuator/caches/{cache}",
			"templated": true
		},
		"health": {
			"href": "http://localhost:8050/actuator/health",
			"templated": false
		},
		"health-path": {
			"href": "http://localhost:8050/actuator/health/{*path}",
			"templated": true
		},
		"info": {
			"href": "http://localhost:8050/actuator/info",
			"templated": false
		},
		"conditions": {
			"href": "http://localhost:8050/actuator/conditions",
			"templated": false
		},
		"configprops": {
			"href": "http://localhost:8050/actuator/configprops",
			"templated": false
		},
		"configprops-prefix": {
			"href": "http://localhost:8050/actuator/configprops/{prefix}",
			"templated": true
		},
		"loggers-name": {
			"href": "http://localhost:8050/actuator/loggers/{name}",
			"templated": true
		},
		"loggers": {
			"href": "http://localhost:8050/actuator/loggers",
			"templated": false
		},
		"heapdump": {
			"href": "http://localhost:8050/actuator/heapdump",
			"templated": false
		},
		"threaddump": {
			"href": "http://localhost:8050/actuator/threaddump",
			"templated": false
		},
		"metrics-requiredMetricName": {
			"href": "http://localhost:8050/actuator/metrics/{requiredMetricName}",
			"templated": true
		},
		"metrics": {
			"href": "http://localhost:8050/actuator/metrics",
			"templated": false
		},
		"scheduledtasks": {
			"href": "http://localhost:8050/actuator/scheduledtasks",
			"templated": false
		},
		"mappings": {
			"href": "http://localhost:8050/actuator/mappings",
			"templated": false
		},
		"refresh": {
			"href": "http://localhost:8050/actuator/refresh",
			"templated": false
		},
		"serviceregistry": {
			"href": "http://localhost:8050/actuator/serviceregistry",
			"templated": false
		}
	}
}
```
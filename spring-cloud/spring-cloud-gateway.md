# SPring cloud gateway
{docsify-updated}

> https://docs.spring.io/spring-cloud-gateway/reference/

## 集成
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway-server-webflux</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway-server-webmvc</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-circuitbreaker-resilience4j</artifactId>
</dependency>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

## 基础概念与原理
+ `Route` ： 网关的基本构建单元。由ID、目标URI、 `Predicate` 集合和 `filters` 集合定义。当 `Predicate` 为真时，路由即匹配成功。
+ `Predicate` :  Java 8 `Predicate` 接口。输入类型为Spring框架的 `ServerWebExchange` 。 可以匹配HTTP请求中的任何内容，例如请求头或参数。
+ `Filter` : 这些是通过特定工厂构造的 `GatewayFilter` 实例。可以在发送请求到下游之前或之后修改请求和响应。
+ `RouteDefinitionLocator` : 根据定义（yml/property文件/consul 配置）加载路由配置信息。
+ `RouteLocator` : 根据配置信息加载路由。

<center><img src="pics/spring_cloud_gateway_diagram.png" alt=""></center>

客户端向Spring Cloud网关发送请求。若网关处理器映射判定请求匹配路由，则将其转发至网关Web处理器。该处理器通过专属过滤器链处理请求。过滤器以点分隔线划分的原因在于：它们可在代理请求发送前、后执行逻辑。所有"预处理"过滤器逻辑执行完毕后，代理请求才会发出。代理请求发出后，将执行"后置"过滤器逻辑。

自动配置类： `GatewayAutoConfiguration` , `GatewayDiscoveryClientAutoConfiguration`

> https://zhuanlan.zhihu.com/p/11306458448

<center><img src="pics/route.png" width="60%"></center>

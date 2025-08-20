# spring cloud loadbancer
{docsify-updated}

> https://docs.spring.io/spring-cloud-commons/reference/spring-cloud-commons/loadbalancer.html  
> https://12factor.net/


## 自定义负载均衡策略
默认使用的 `ReactiveLoadBalancer` 实现是 `RoundRobinLoadBalancer` 。要为选定的服务或所有服务切换到不同的实现，可以使用自定义负载平衡器配置机制。
```
public class CustomLoadBalancerConfiguration {

	@Bean
	ReactorLoadBalancer<ServiceInstance> randomLoadBalancer(Environment environment,
			LoadBalancerClientFactory loadBalancerClientFactory) {
		String name = environment.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);
		return new RandomLoadBalancer(loadBalancerClientFactory
				.getLazyProvider(name, ServiceInstanceListSupplier.class),
				name);
	}
}
```

## 与各种客户端集成使用
> https://docs.spring.io/spring-cloud-commons/reference/spring-cloud-commons/common-abstractions.html#rest-template-loadbalancer-client

为了便于使用 Spring Cloud LoadBalancer，我们提供了 `ReactorLoadBalancerExchangeFilterFunction` （可与 `WebClient` 一起使用）和 `BlockingLoadBalancerClient` （可与 `RestTemplate` 和 `RestClient` 一起使用）。使用示例：
+ [Spring RestTemplate as a LoadBalancer Client](https://docs.spring.io/spring-cloud-commons/reference/spring-cloud-commons/common-abstractions.html#rest-template-loadbalancer-client)
+ [Spring RestClient as a LoadBalancer Client](https://docs.spring.io/spring-cloud-commons/reference/spring-cloud-commons/common-abstractions.html#rest-client-loadbalancer-client)
+ [Spring WebClient as a LoadBalancer Client](https://docs.spring.io/spring-cloud-commons/reference/spring-cloud-commons/common-abstractions.html#webclinet-loadbalancer-client)
+ [Spring WebFlux WebClient with ReactorLoadBalancerExchangeFilterFunction](https://docs.spring.io/spring-cloud-commons/reference/spring-cloud-commons/common-abstractions.html#webflux-with-reactive-loadbalancer)


## 缓存
除了每次需要选择实例时通过 `DiscoveryClient` 获取实例的基本 `ServiceInstanceListSupplier` 实现外，我们还提供了两种缓存实现。

### 基于 Caffine 缓存的实现
如果 classpath 中包含 `com.github.ben-manes.caffeine:caffeine` ，则将使用基于 `Caffeine` 的实现。 `LoadBalancerCacheConfiguration` 可以用来配置。

### 默认的实现
如果 classpath 中没有 Caffeine，则将使用 Spring-cloud-starter-loadbalancer 自动提供的 `DefaultLoadBalancerCache` 。也是通过 `LoadBalancerCacheConfiguration` 配置

## 权重缓存配置
```
public class CustomLoadBalancerConfiguration {

	@Bean
	public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
			ConfigurableApplicationContext context) {
		return ServiceInstanceListSupplier.builder()
					.withDiscoveryClient()
					.withWeighted(instance -> ThreadLocalRandom.current().nextInt(1, 101))
					.withCaching()
					.build(context);
	}
}
```

## 基于 Zone 的配置
为实现基于区域的负载平衡，我们提供了 `ZonePreferenceServiceInstanceListSupplier` 。首先我们必须为客户端 `DiscoveryClient` 配置特定的区域 zone（例如 `eureka.instance.metadata-map.zone`），然后负载均衡会用这个值过滤可用服务实例，服务实例也必须配置 zone 参数。

`ZonePreferenceServiceInstanceListSupplier` 会过滤检索到的实例，只返回同一区域内的实例。如果区域为空或同一区域内没有实例，则会返回所有检索到的实例。

```
public class CustomLoadBalancerConfiguration {

	@Bean
	public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
			ConfigurableApplicationContext context) {
		return ServiceInstanceListSupplier.builder()
					.withDiscoveryClient()
                    .withCaching()
					.withZonePreference()
					.build(context);
	}
}
```

## 健康检查
`HealthCheckServiceInstanceListSupplier` 提供了健康检查相关的功能与配置。

```
public class CustomLoadBalancerConfiguration {

	@Bean
	public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
			ConfigurableApplicationContext context) {
		return ServiceInstanceListSupplier.builder()
					.withDiscoveryClient()
					.withHealthChecks()
					.build(context);
	    }
}
```

## 自定义配置
使用 `@LoadBalancerClient` 自定义设置负载均衡客户端， `value` 指定要要调用的服务实例（服务发现）， `configuration` 指定自定义的配置类。
```
@Configuration
@LoadBalancerClients({@LoadBalancerClient(value = "stores", configuration = StoresLoadBalancerClientConfiguration.class), @LoadBalancerClient(value = "customers", configuration = CustomersLoadBalancerClientConfiguration.class)})
public class MyConfiguration {

	@Bean
	@LoadBalanced
	public WebClient.Builder loadBalancedWebClientBuilder() {
		return WebClient.builder();
	}
}
```
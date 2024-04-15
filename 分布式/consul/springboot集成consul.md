#  Springboot 集成 Consul
{docsify-updated}

> https://cloud.spring.io/spring-cloud-static/Greenwich.M3/multi/multi_spring-cloud-consul-discovery.html

- [Springboot 集成 Consul](#springboot-集成-consul)
  - [配置类](#配置类)
  - [服务注册过程](#服务注册过程)
  - [服务发现](#服务发现)

consul提供的各种语言的SDK/开发库： https://developer.hashicorp.com/consul/api-docs/libraries-and-sdks

spring-cloud-starter-consul-discovery 使用的是  [consul-api](https://github.com/Ecwid/consul-api) ,依赖其底层的 `ConsulClient` 与 Consul 交互。

### 配置类

```
public class ConsulAutoServiceRegistrationAutoConfiguration {  
  
    @Autowired  
    AutoServiceRegistrationProperties autoServiceRegistrationProperties;  
  
    @Bean  
    @ConditionalOnMissingBean    
    public ConsulAutoServiceRegistration consulAutoServiceRegistration(ConsulServiceRegistry registry,  
          AutoServiceRegistrationProperties autoServiceRegistrationProperties, ConsulDiscoveryProperties properties,  
          ConsulAutoRegistration consulRegistration) {  
       return new ConsulAutoServiceRegistration(registry, autoServiceRegistrationProperties, properties,  
             consulRegistration);  
    }  
  
    @Bean  
    public ConsulAutoServiceRegistrationListener consulAutoServiceRegistrationListener(  
          ConsulAutoServiceRegistration registration) {  
       return new ConsulAutoServiceRegistrationListener(registration);  
    }  
  
    @Bean  
    @ConditionalOnMissingBean    
    public ConsulAutoRegistration consulRegistration(  
          AutoServiceRegistrationProperties autoServiceRegistrationProperties, ConsulDiscoveryProperties properties,  
          ApplicationContext applicationContext,  
          ObjectProvider<List<ConsulRegistrationCustomizer>> registrationCustomizers,  
          ObjectProvider<List<ConsulManagementRegistrationCustomizer>> managementRegistrationCustomizers,  
          HeartbeatProperties heartbeatProperties) {  
       return ConsulAutoRegistration.registration(autoServiceRegistrationProperties, properties, applicationContext,  
             registrationCustomizers.getIfAvailable(), managementRegistrationCustomizers.getIfAvailable(),  
             heartbeatProperties);  
    }
```
1. 如果要禁用服务发现（当你只作为服务提供者时），设置`spring.cloud.consul.discovery.enabled=false`，默认是 true
2. 如果要禁用服务注册（当你只作为服务消费者时），设置`spring.cloud.consul.discovery.register=fales`，默认是 true
3. 配置Consul健康检查，conusl发请求给服务以检查健康状态
   Consul 实例的健康检查默认位于"/actuator/health"，这是 Spring Boot Actuator 应用程序中健康端点的默认位置。如果使用非默认上下文路径或 servlet 路径（如 server.servletPath=/foo），即使是 Actuator 应用程序也需要更改。
   ```
   spring:
     cloud:
       consul:
         discovery:
           healthCheckPath: ${management.server.servlet.context-path}/actuator/health
           healthCheckInterval: 15s
   ```
   如果要禁用健康检查，设置 `spring.cloud.consul.discovery.register-health-check=false`,`management.health.consul.enabled=false`
4. TTL检查：应用程序会主动向 Consul 代理发送心跳信号，而不是 Consul 代理向应用程序发送请求。
   ```
   spring:
     cloud:
       consul:
         discovery:
           heartbeat:
             enabled: true
             ttl: 10s
   ```
5. 注册服务时提供 MetaData
   ```
   spring:
     cloud:
       consul:
         discovery:
           metadata:
             myfield: myvalue
             anotherfield: anothervalue
   ```

### 服务注册过程

+ `ConsulDiscoveryProperties`: springboot配置属性加载类,加载`spring.cloud.consul.discovery`前缀的配置，用于配置Consul服务发现与注册的配置
+ `ConsulAutoRegistration`: 封装需要注册的服务的信息，例如服务名、服务端口、健康检查、心跳等，它的 `registration` 方法会生成需要向 consul 注册的服务的信息（ `NewService` ）。
      ```
      public static ConsulAutoRegistration registration(
			AutoServiceRegistrationProperties autoServiceRegistrationProperties, ConsulDiscoveryProperties properties,
			ApplicationContext context, List<ConsulRegistrationCustomizer> registrationCustomizers,
			List<ConsulManagementRegistrationCustomizer> managementRegistrationCustomizers,
			HeartbeatProperties heartbeatProperties) {

		NewService service = new NewService();
		String appName = getAppName(properties, context.getEnvironment());
		service.setId(getInstanceId(properties, context));
		if (!properties.isPreferAgentAddress()) {
			service.setAddress(properties.getHostname());
		}
		service.setName(normalizeForDns(appName));
		service.setTags(new ArrayList<>(properties.getTags()));
		service.setEnableTagOverride(properties.getEnableTagOverride());
		service.setMeta(getMetadata(properties));

		if (properties.getPort() != null) {
			service.setPort(properties.getPort());
			// we know the port and can set the check
			setCheck(service, autoServiceRegistrationProperties, properties, context, heartbeatProperties);
		}

		ConsulAutoRegistration registration = new ConsulAutoRegistration(service, autoServiceRegistrationProperties,
				properties, context, heartbeatProperties, managementRegistrationCustomizers);
		customize(registrationCustomizers, registration);
		return registration;
	}
      ```
+ `ConsulAutoServiceRegistration` : 封装服务注册过程的类，含有一个 `ConsulServiceRegistry` 属性，该类能真正使用consul sdk中的能力向 consul 注册服务
+ `ConsulServiceRegistry`： 真正与 consul 服务交互实施服务注册( `register`方法 )与卸载（`deregister`方法）的类。
      ```
      public class ConsulServiceRegistry implements ServiceRegistry<ConsulRegistration> {
            private final ConsulClient client;
            private final ConsulDiscoveryProperties properties;
            private final TtlScheduler ttlScheduler;
            private final HeartbeatProperties heartbeatProperties;

            public ConsulServiceRegistry(ConsulClient client, ConsulDiscoveryProperties properties, TtlScheduler ttlScheduler,
                        HeartbeatProperties heartbeatProperties) {
                  this.client = client;
                  this.properties = properties;
                  this.ttlScheduler = ttlScheduler;
                  this.heartbeatProperties = heartbeatProperties;
            }

            @Override
            public void register(ConsulRegistration reg) {
                  log.info("Registering service with consul: " + reg.getService());
                  try {
                        this.client.agentServiceRegister(reg.getService(), this.properties.getAclToken());
                        NewService service = reg.getService();
                        if (this.heartbeatProperties.isEnabled() && this.ttlScheduler != null && service.getCheck() != null
                                    && service.getCheck().getTtl() != null) {
                              this.ttlScheduler.add(reg.getService());
                        }
                  }
                  catch (ConsulException e) {
                        if (this.properties.isFailFast()) {
                              log.error("Error registering service with consul: " + reg.getService(), e);
                              ReflectionUtils.rethrowRuntimeException(e);
                        }
                        log.warn("Failfast is false. Error registering service with consul: " + reg.getService(), e);
                  }
            }

            @Override
            public void deregister(ConsulRegistration reg) {
                  if (this.ttlScheduler != null) {
                        this.ttlScheduler.remove(reg.getInstanceId());
                  }
                  if (log.isInfoEnabled()) {
                        log.info("Deregistering service with consul: " + reg.getInstanceId());
                  }
                  this.client.agentServiceDeregister(reg.getInstanceId(), this.properties.getAclToken());
            }
      }
      ```
+ `ConsulAutoServiceRegistrationListener`：服务注册的触发者
      ```
      public class ConsulAutoServiceRegistrationListener implements SmartApplicationListener {

            private final ConsulAutoServiceRegistration autoServiceRegistration;

            @Override
            public void onApplicationEvent(ApplicationEvent applicationEvent) {
                  if (applicationEvent instanceof WebServerInitializedEvent) {
                        WebServerInitializedEvent event = (WebServerInitializedEvent) applicationEvent;

                        ApplicationContext context = event.getApplicationContext();
                        if (context instanceof ConfigurableWebServerApplicationContext) {
                              if ("management".equals(((ConfigurableWebServerApplicationContext) context).getServerNamespace())) {
                                    return;
                              }
                        }
                        this.autoServiceRegistration.setPortIfNeeded(event.getWebServer().getPort());
                        this.autoServiceRegistration.start();
                  }
            }
      }
      ```
      当收到Springboot的`WebServerInitializedEvent` 事件时，就会开始注册服务。

### 服务发现

1. 使用 `Spring Cloud LoadBalancer`
Spring Cloud 支持 Feign（REST 客户端构建器）和 Spring RestTemplate，以便使用逻辑服务名称/ID 而不是物理 URL 查找服务。Feign 和具有 RestTemplate 都利用 Spring Cloud LoadBalancer 实现客户端负载平衡。
2. 使用 `DiscoveryClient`
   ```
   @Autowired
   private DiscoveryClient discoveryClient;
   
   public String serviceUrl() {
       List<ServiceInstance> list = discoveryClient.getInstances("STORES");
       if (list != null && list.size() > 0 ) {
           return list.get(0).getUri();
       }
       return null;
   }
   ```
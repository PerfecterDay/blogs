
consul提供的各种语言的SDK/开发库： https://developer.hashicorp.com/consul/api-docs/libraries-and-sdks

spring-cloud-starter-consul-discovery 使用的是  [consul-api](https://github.com/Ecwid/consul-api) ,依赖其底层的 `ConsulClient` 与 Consul 交互。

### ConsulAutoServiceRegistrationAutoConfiguration - 自动配置类
```
public class ConsulAutoServiceRegistrationAutoConfiguration {  
  
    @Autowired  
    AutoServiceRegistrationProperties autoServiceRegistrationProperties;  
  
    @Bean  
    @ConditionalOnMissingBean    public ConsulAutoServiceRegistration consulAutoServiceRegistration(ConsulServiceRegistry registry,  
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
    @ConditionalOnMissingBean    public ConsulAutoRegistration consulRegistration(  
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
ConsulDiscoveryProperties - springboot配置属性加载类
ConsulAutoRegistration

ConsulDiscoveryClientConfiguration
            ConsulDiscoveryProperties
HeartbeatProperties

ConsulAutoServiceRegistrationListener
只有web应用时才会注册服务
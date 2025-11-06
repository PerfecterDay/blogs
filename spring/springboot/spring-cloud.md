# Spring Cloud Commons
{docsify-updated}

> https://docs.spring.io/spring-cloud-commons/reference/index.html


`Spring Cloud Context` 为 Spring Cloud 应用程序的 ApplicationContext 提供实用工具和特殊服务（bootstrap context, encryption, refresh scope, and environment endpoints）。 `Spring Cloud Commons` 是一组抽象层和通用类，用于不同 Spring Cloud 实现（如 Spring Cloud Netflix 和 Spring Cloud Consul）。

## Spring Cloud Context
Spring Boot 对如何使用 Spring 构建应用程序有着明确的见解。例如，它为常见配置文件提供了约定位置，并为常规管理和监控任务提供了端点。Spring Cloud 在此基础上构建，并添加了若干功能，这些功能是系统中许多组件会使用或偶尔需要的。

### Environment Changes
应用程序监听 `EnvironmentChangeEvent` 事件，并通过几种标准方式响应环境变更（如使用 `@Bean` 添加一个 `ApplicationListeners`）。当检测到 `EnvironmentChangeEvent` 时，该事件包含已变更的键值列表，应用程序利用这些信息来：
+ 重新绑定上下文中的任何 `@ConfigurationProperties` bean。
+ 为 `logging.level.*` 下的任何属性设置日志记录级别。

请注意，Spring Cloud Config Client 默认不会轮询 `Environment` 中的变更。通常我们不建议采用这种方式检测变更（尽管您可以通过 `@Scheduled` 注解进行配置）。若您拥有分布式的应用实例，更优方案是向所有实例广播 `EnvironmentChangeEvent` 事件，而非让它们轮询变更（例如通过 Spring Cloud Bus 实现）。

`EnvironmentChangeEvent` 涵盖了大量的属性更新应用场景，只要您能够实际修改 `Environment` 并发布 `EnvironmentChangeEvent` 事件即可。请注意这些 API 是公开的，且属于 Spring 核心组件。然后就可通过访问 `/configprops` 端点（Spring Boot Actuator 的标准功能）验证变更是否已绑定至 `@ConfigurationProperties` bean。例如， `DataSource` 的 `maxPoolSize` 可在运行时动态调整（Spring Boot 默认创建的 DataSource 即为 `@ConfigurationProperties` Bean），从而实现容量动态扩展。

测试代码：
```
@Resource
ConfigurableEnvironment environment;

@Resource
ApplicationContext applicationContext;

@GetMapping("/aaa")
public String aaa() {
    Map<String, Object> map = new HashMap<>();
    map.put("cms.url", "new-value"); // 修改或新增属性

    // 添加到最前面，优先级最高
    MapPropertySource propertySource = new MapPropertySource("dynamic-properties", map);
    environment.getPropertySources().addFirst(propertySource);

    HashSet set = new HashSet();
    set.add("cms.url");
    applicationContext.publishEvent(new EnvironmentChangeEvent(set));
    return environment.getProperty("cms.url");
}
```

访问 `http://localhost:8091/actuator/configprops` 可以看到 new-value 的值。

但是重新绑定 `@ConfigurationProperties` 无法覆盖另一类重要用例场景：当您需要更精细地控制刷新机制，或要求变更在整个 `ApplicationContext` 中具有原子性时。

例如上边的修改 `cms.url` 的例子，当事件发布后，`environment.getProperty("cms.url")` 代码和 `@ConfigurationProperties` 注解的配置 bean 都会得到新值。但是，如果有个bean 定义：
```
@Component
public class MyService {
    @Value("${cms.url}")
    private String url;
}
```
此时的 url 不会更新成新值，因为 @Value 的注入是在 Bean 创建时完成的，Spring 不会因为 Environment 变化就重新注入 Bean 的字段。

为解决这些需求，我们引入了 `@RefreshScope` 注解。

### Refresh Scope
当配置发生变更时，标记为 `@RefreshScope` 的Spring `@Bean` 会获得特殊处理。此特性解决了有状态 `Bean` 仅在初始化时才注入配置的问题。例如，当通过环境变量修改数据库URL时，若数据源存在已打开的连接，通常需要允许这些连接持有者完成当前操作。但是，下次从连接池借用连接时，即可获得使用新URL的连接。

有时，对于某些只能初始化一次的Bean，甚至必须应用 `@RefreshScope` 注解。如果Bean是“不可变”的，则必须为其添加 `@RefreshScope` 注解，或在 `spring.cloud.refresh.extra-refreshable` 属性键下指定类名。

Refresh scope beans是惰性代理，仅在使用时（即方法被调用时）初始化，该作用域充当初始化值的缓存。若需强制Bean在下次方法调用时重新初始化，必须使其缓存条目失效。

`RefreshScope` 是上下文中的一个 Bean，其公开的 `refreshAll()` 方法通过清除目标缓存来刷新作用域内的所有 `Bean` 。 `/refresh` 端点通过 HTTP 或 JMX 协议暴露此功能。若需按名称刷新单个 Bean，还可使用 `refresh(String)` 方法。

要暴露 `/refresh` 端点，您需要在应用程序中添加以下配置：
```
management:
  endpoints:
    web:
      exposure:
        include: refresh
```

refresh 端点： `curl -XPOST http://localhost:8091/actuator/refresh`


测试实例：
```
@Component
@RefreshScope
@Data
public class B {

    @Value("${cms.url}")
    private String cmsUrl;
}

@Resource
    B b;

@GetMapping("/aaa")
public String aa(){
    Map<String, Object> map = new HashMap<>();
    map.put("cms.url", "new-value"); // 修改或新增属性

    // 添加到最前面，优先级最高
    MapPropertySource propertySource = new MapPropertySource("dynamic-properties", map);
    environment.getPropertySources().addFirst(propertySource);

    HashSet set = new HashSet();
    set.add("cms.url");
    applicationContext.publishEvent(new EnvironmentChangeEvent(set));
    return b.getCmsUrl();
}
```

当你执行 `curl http://localhost:8050/user/aaa` 时，即使刷新了配置值，依旧返回的是初始化时的值。只有执行了 `curl -XPOST http://localhost:8091/actuator/refresh` 后，再次执行 `curl http://localhost:8050/user/aaa` 才会返回新值。


### 原理
`BeanFactory` 在生产 bean 时，会根据 beandefinition 的信息来获取对应的 scope，不同的 scope 使用不同的 `Scope` 实现类来生成 bean 对象，一些常见的 `Scope` 实现类如下：
<center><img src="pics/scope.png" alt=""></center>

```
AbstractBeanFactory -> doGetBean :
Object scopedInstance = scope.get(beanName, () -> {
    beforePrototypeCreation(beanName);
    try {
        return createBean(beanName, mbd, args);
    }
    finally {
        afterPrototypeCreation(beanName);
    }
});
```

其中 `RefreshScope` 继承自 `GenericScope` ，其 `get` 方法如下：
```
public Object get(String name, ObjectFactory<?> objectFactory) {
    BeanLifecycleWrapper value = this.cache.put(name, new BeanLifecycleWrapper(name, objectFactory));
    this.locks.putIfAbsent(name, new ReentrantReadWriteLock());
    try {
        return value.getBean();
    }
    catch (RuntimeException e) {
        this.errors.put(name, e);
        throw e;
    }
}
```
Cache 结构如下：
<center><img src="pics/refresh_scope.png" alt=""></center>

真正实现 `@RefreshScope` bean 创建的是 `BeanLifecycleWrapper` ，其中会保存 bean 实例对象，如果实例对象已经创建就直接返回该对象，如果没有创建就会创建一个新的实例对象。

```
RefreshAutoConfiguration
RefreshEventListener
RefreshScopeRefreshedEvent
```

## Spring Cloud Commons
诸如服务发现、负载均衡和熔断限流等模式，天然适合构建一个通用抽象层，该层可被所有Spring Cloud客户端使用，且与具体实现无关。（比如使用 Eurika 或者 Consul 都可以实现服务发现）。


### `@EnableDiscoveryClient`
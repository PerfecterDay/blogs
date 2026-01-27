# 自定义 Bean 特性
{docsify-updated}

Spring框架提供了多种接口，可用于自定义Bean的特性。本节将这些接口按以下方式分组：
+ Lifecycle Callbacks
+ `ApplicationContextAware` and `BeanNameAware`
+ Other `Aware` Interfaces

## Lifecycle Callbacks
要与容器对 Bean 生命周期的管理进行交互，可以实现 Spring 的 `InitializingBean` 和 `DisposableBean` 接口。容器会在初始化时调用前者（ `InitializingBean` ）的 `afterPropertiesSet()` 方法，在销毁时调用后者（ `DisposableBean` ）的 `destroy()` 方法，从而让 Bean 在初始化和销毁时执行特定操作。

在现代Spring应用中，使用 `JSR-250` 的 `@PostConstruct` 和 `@PreDestroy` 注解通常被视为接收生命周期回调的最佳实践。采用这些注解可以使Bean无需与Spring专属接口耦合。

在内部，Spring框架通过 `BeanPostProcessor` 来处理所有可识别的回调接口，并调用相应的方法。若需实现Spring默认未提供的自定义功能或其他生命周期行为，开发者可自行实现 `BeanPostProcessor` 。

除了 bean 本身初始化和销毁回调外，Spring管理的对象还可以实现 `Lifecycle` 接口，使得 bean 能够参与到容器的生命周期的特定阶段处理逻辑。


### Initialization 回调
`org.springframework.beans.factory.InitializingBean` 接口允许容器在为 Bean 设置所有必要属性后，让 Bean 执行初始化工作。 `InitializingBean` 接口定义了一个方法：
```
void afterPropertiesSet() throws Exception;
```
建议不要使用 `InitializingBean` 接口，因为它会使代码与Spring产生耦合。替代方案是:
1. 使用 `@PostConstruct` 注解
2. 或指定POJO初始化方法
   1. 对于基于XML的配置元数据，可通过 `init-method` 属性指定具有 **`void` 无参签名的方法名称**。
   2. 使用 java 配置时，使用 `@Bean` 注解的 `initMethod` 属性

以下几种配置方式能达到相同的效果：
```
public class ExampleBean implements InitializingBean{
    @Override
	public void afterPropertiesSet() {
		// do some initialization work
	}
}

public class ExampleBean{
    @PostConstruct
    void init(){
        ...
    }
}


<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>

@Configuration
public class AppConfig {

	@Bean(initMethod = "init")
	public ExampleBean beanOne() {
		return new ExampleBean();
	}
}
```
当直接使用Java 编程配置时，可以随心所欲地操作对象，而无需始终依赖容器的生命周期管理。

请注意，@PostConstruct 注解方法及常规初始化方法均在容器的单例创建锁内执行。只有当 @PostConstruct 方法返回后，bean 实例才被视为完全初始化完毕并准备好发布给其他组件。此类独立初始化方法仅用于验证配置状态，并可能根据给定配置准备某些数据结构，但不得进行任何涉及外部 bean 访问的操作。否则将存在初始化死锁风险。

### Destruction 回调
实现 `org.springframework.beans.factory.DisposableBean` 接口可使 bean 在包含它的容器被销毁时获得回调。 `DisposableBean` 接口定义了一个方法：
```
void destroy() throws Exception;
```

类似地，我们推荐使用 `@PreDestroy` 注解，或者在 XML 中使用 `<bean/>` 元素的 `destroy-method` 属性。当使用 java 配置时，使用 `@Bean` 注解的 `destroyMethod` 属性。
```
public class AnotherExampleBean implements DisposableBean {
	@Override
	public void destroy() {
		// do some destruction work (like releasing pooled connections)
	}
}


public class AnotherExampleBean {
	@PreDestroy
	public void preDestroy() {
		// do some destruction work (like releasing pooled connections)
	}
}

<bean id="exampleDestructionBean" class="examples.ExampleBean" destroy-method="cleanup"/>
public class AnotherExampleBean {
	public void cleanup() {
		// do some destruction work (like releasing pooled connections)
	}
}


@Configuration
public class AppConfig {

	@Bean(destroyMethod = "cleanup")
	public ExampleBean beanOne() {
		return new ExampleBean();
	}
}
```

默认情况下，使用Java配置定义且具有 `public close` 或 `shutdown` 方法的Bean会自动关联一个销毁回调。若定义了 `public close` 或 `shutdown` 方法，但不希望容器关闭时调用该方法，可在Bean定义中添加 `@Bean(destroyMethod = "")` 注解以禁用默认（推断）模式。


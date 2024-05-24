# ImportBeanDefinitionRegistrar 自定义控制 bean 定义
{docsify-updated}

```
public interface ImportBeanDefinitionRegistrar {
	/**
	 * Register bean definitions as necessary based on the given annotation metadata of
	 * the importing {@code @Configuration} class.
	 */
	default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry,
			BeanNameGenerator importBeanNameGenerator) {

		registerBeanDefinitions(importingClassMetadata, registry);
	}

	default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
	}
}
```

由源码可以看出， `ImportBeanDefinitionRegistrar` 本质上是一个接口。在 `ImportBeanDefinitionRegistrar` 接口中，有一个 `registerBeanDefinitions()` 方法，通过 `registerBeanDefinitions()` 方法，我们可以向Spring容器中注册bean实例。

Spring官方在动态注册bean时(大多数时候是为一些特殊注解生成代理 bean)，大部分套路其实是使用 `ImportBeanDefinitionRegistrar` 接口。

所有实现了该接口的类都会被 `ConfigurationClassPostProcessor` 处理， `ConfigurationClassPostProcessor` 实现了 `BeanFactoryPostProcessor` 接口，所以 `ImportBeanDefinitionRegistrar` 中动态注册的bean是优先于依赖其的bean初始化的，也能被aop、validator等机制处理。
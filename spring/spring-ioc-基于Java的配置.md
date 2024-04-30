#  Spring IOC 基于Java的配置
 {docsify-updated}
> https://docs.spring.io/spring-framework/reference/core/beans/java.html


## 注解 @Import
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {
	/**
	 * {@link Configuration @Configuration}, {@link ImportSelector},
	 * {@link ImportBeanDefinitionRegistrar}, or regular component classes to import.
	 */
	Class<?>[] value();
}
```
假如项目中依赖了一个第三方包，这个包中有些 `@Configuration` 配置类没有在我们的扫描路径上，而我们又需要使用这些配置，那么可以在我们自己的配置类上使用 `@Import` 注解导入第三方的配置类。
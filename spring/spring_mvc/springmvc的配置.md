#  Spring MVC 配置
{docsify-updated}

- [Spring MVC 配置](#spring-mvc-配置)
	- [WebMvcConfigurer](#webmvcconfigurer)
	- [Springboot 的自动配置- WebMvcAutoConfiguration](#springboot-的自动配置--webmvcautoconfiguration)

应用程序可以声明处理请求所需的特殊 Bean 类型（`HandlerMapping、HandlerAdapter、HandlerExceptionResolver、ViewResolver`等）。 `DispatcherServlet` 会在 `WebApplicationContext` 中检查每个特殊 Bean。如果没有匹配的 Bean 类型，它就会使用classpath下的 `DispatcherServlet.properties` 中列出的默认类型配置。

## WebMvcConfigurer
在Java配置中，可以使用 `@EnableWebMvc` 注解并实现 `WebMvcConfigurer` 接口来启用MVC配置，，如下例所示：

```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
	// Implement configuration methods...
}


@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}
```

`@EnableWebMvc` 注解的功能大多是通过 `DelegatingWebMvcConfiguration` 配置类来实现的，它继承自 `WebMvcConfigurationSupport`，默认会注入所有的 `WebMvcConfigurer` 。
```
@Configuration(proxyBeanMethods = false)
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {

	private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();


	@Autowired(required = false)
	public void setConfigurers(List<WebMvcConfigurer> configurers) {
		if (!CollectionUtils.isEmpty(configurers)) {
			this.configurers.addWebMvcConfigurers(configurers);
		}
	}
....
}
```
在 `WebMvcConfigurationSupport` 中有很多声明了 MVC 相关的 bean 的 `@Bean` 方法 ，这些声明 bean 的方法中会调用相应的 `configureXXX`/`addXXX` 方法，这些方法最终会去调用注入的 `WebMvcConfigurer` 相关的方法。这就是为什么我们只要写一个实现 `WebMvcConfigurer` 接口的类并注入到容器中，就能起作用的原因。

`WebMvcConfigurer` 接口中的一些方法：

+ `default void configurePathMatch(PathMatchConfigurer configurer) {} ` ：自定义Path匹配逻辑
+ `default void configureContentNegotiation(ContentNegotiationConfigurer configurer) {}` ：自定义内容协商
+ `default void configureAsyncSupport(AsyncSupportConfigurer configurer) {}` ：自定义异步支持配置
+ `default void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {}` ：
+ `default void addFormatters(FormatterRegistry registry) {}` ：添加自定义格式化器
+ `default void addInterceptors(InterceptorRegistry registry) {}` ：添加 Interceptor
+ `default void addResourceHandlers(ResourceHandlerRegistry registry) {}` ：
+ `default void addCorsMappings(CorsRegistry registry) {}` ：添加 CORS 跨域配置
+ `default void addViewControllers(ViewControllerRegistry registry) {}` ：
+ `default void configureViewResolvers(ViewResolverRegistry registry) {}` ：配置 ViewResolver 
+ `default void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {}` ：配置 HandlerMethodArgumentResolver
+ `default void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> handlers) {}` ：
+ `default void configureMessageConverters(List<HttpMessageConverter<?>> converters) {}` ：配置 HttpMessageConverter
+ `default void extendMessageConverters(List<HttpMessageConverter<?>> converters) {}` ：
+ `default void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {}` ：配置 HandlerExceptionResolver
+ `default void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {}` ：

## Springboot 的自动配置- WebMvcAutoConfiguration
Springboot 中使用 `WebMvcAutoConfiguration` 实现自动配置，在其中定义了 `EnableWebMvcConfiguration` 内部类，这个类实现了和 `@EnableWebMvc` 相同的功能。
```
// Configuration equivalent to @EnableWebMvc.
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(WebProperties.class)
public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration implements ResourceLoaderAware{...}
```
这就是为什么在 Springboot 中没有使用 `@EnableWebMvc` 注解，依然能启用 mvc 功能的原因。


替换掉默认的 tomcat 容器：
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```
2024-04-29 13:41:25.078 [main] INFO  com.gtja.gjyw.UserCenterApp  - Started UserCenterApp in 50.119 seconds (JVM running for 51.346)

2024-04-29 17:42:40.493 [main] INFO  com.gtja.gjyw.UserCenterApp  - Started UserCenterApp in 32.094 seconds (JVM running for 33.44) undertow
2024-04-29 17:43:50.493 [main] INFO  com.gtja.gjyw.UserCenterApp  - Started UserCenterApp in 32.223 seconds (JVM running for 33.654) jetty
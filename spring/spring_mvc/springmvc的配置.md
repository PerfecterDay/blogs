#  Spring MVC 配置
{docsify-updated}

- [Spring MVC 配置](#spring-mvc-配置)
	- [WebMvcConfigurer](#webmvcconfigurer)
	- [Springboot 的自动配置- WebMvcAutoConfiguration](#springboot-的自动配置--webmvcautoconfiguration)
	- [替换掉默认的 tomcat 容器](#替换掉默认的-tomcat-容器)
	- [Springboot 内置容器配置](#springboot-内置容器配置)
		- [Springboot中的内置容器启动与配置](#springboot中的内置容器启动与配置)

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


## 替换掉默认的 tomcat 容器
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
		<!-- Exclude the Tomcat dependency -->
		<exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<!-- Use Jetty instead -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```
2024-04-29 13:41:25.078 [main] INFO  com.gtja.gjyw.UserCenterApp  - Started UserCenterApp in 50.119 seconds (JVM running for 51.346)
2024-04-29 17:42:40.493 [main] INFO  com.gtja.gjyw.UserCenterApp  - Started UserCenterApp in 32.094 seconds (JVM running for 33.44) undertow
2024-04-29 17:43:50.493 [main] INFO  com.gtja.gjyw.UserCenterApp  - Started UserCenterApp in 32.223 seconds (JVM running for 33.654) jetty

## Springboot 内置容器配置
> https://docs.spring.io/spring-boot/docs/2.0.9.RELEASE/reference/html/howto-embedded-web-servers.html

### Springboot中的内置容器启动与配置
Sprinboot 使用代码编程的方式启动内置的 Servlet 容器，通过 `TomcatServletWebServerFactory/JettyServletWebServerFactory/UndertowServletWebServerFactory` 等类实现。Springboot 启动内置 Servlet 的具体过程如下：
1. 创建一个继承自 `ServletWebServerApplicationContext` 的 `AnnotationConfigServletWebServerApplicationContext` 
2. 在 `ServletWebServerApplicationContext#createWebServer()` 方法中创建内置的 servlet 容器，使用 `ServletWebServerFactory#getWebServer(ServletContextInitializer... initializers)` 工厂接口方法来创建，可以传入 `ServletContextInitializer` 来配置 servlet
3. `ServletWebServerApplicationContext#createWebServer()` 在调用 `ServletWebServerFactory#getWebServer(ServletContextInitializer... initializers)` 方法时，传入了自定义的 `ServletContextInitializer`  

    <img src="pics/springboot-embed.png" alt="" />

	```java
	private org.springframework.boot.web.servlet.ServletContextInitializer getSelfInitializer() {
		return this::selfInitialize;
	}

	private void selfInitialize(ServletContext servletContext) throws ServletException {
		prepareWebApplicationContext(servletContext);
		registerApplicationScope(servletContext);
		WebApplicationContextUtils.registerEnvironmentBeans(getBeanFactory(), servletContext);
		for (ServletContextInitializer beans : getServletContextInitializerBeans()) {
			beans.onStartup(servletContext);
		}
	}
	```
	在 `selfInitialize` 方法中，会去遍历调用 `ServletContextInitializer#onStartup(ServletContext servletContext)` 方法。
4. 在 `ServletContextInitializer` <- `RegistrationBean`<-`DynamicRegistrationBean`<-`ServletRegistrationBean`<-`DispatcherServletRegistrationBean` 的体系下，只要我们声明 `DispatcherServletRegistrationBean` 或者 其他的 `RegistrationBean` 类型，springboot 就会帮我们注册到 servlet 容器。
5. springboot 的 `DispatcherServletAutoConfiguration#DispatcherServletRegistrationConfiguration#dispatcherServletRegistration(DispatcherServlet dispatcherServlet,WebMvcProperties webMvcProperties, ObjectProvider<MultipartConfigElement> multipartConfig)`方法就声明了`DispatcherServletRegistrationBean`，这就是为什么Springboot能自动帮我们配置好 DispatcherServlet 的原因。

通过以上分析，如果我们想注册除了 `DispatcherServlet` 以外的自定义 servlet ，只要声明一个 `ServletRegistrationBean` 的 bean 即可。类似的，`FilterRegistrationBean` 可以注册自定义的 `Filter` 。 如果自己实现 `Filetr` 接口又想使用 Spring 容器功能，springboot 提供了方便的 `DelegatingFilterProxyRegistrationBean` 类型，我们只要自定义一个 `DelegatingFilterProxyRegistrationBean` 类型的 bean 即可。

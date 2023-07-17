## Spring MVC 配置
{docsify-updated}

- [Spring MVC 配置](#spring-mvc-配置)
	- [WebMvcConfigurer](#webmvcconfigurer)



### WebMvcConfigurer
在Java配置中，可以使用 `@EnableWebMvc` 注解并实现 `WebMvcConfigurer` 接口来启用MVC配置，，如下例所示：

```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
	// Implement configuration methods...
}
```
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







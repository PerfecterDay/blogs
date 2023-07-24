## spring-cloud-openfeign
{docsify-updated}
> https://www.baeldung.com/spring-cloud-openfeign
> https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/

- [spring-cloud-openfeign](#spring-cloud-openfeign)
	- [简介](#简介)
	- [集成步骤](#集成步骤)
	- [配置](#配置)
		- [Java 代码配置](#java-代码配置)
		- [配置文件配置](#配置文件配置)
		- [Interceptors](#interceptors)


### 简介
Spring Cloud OpenFeign 是一个用于Spring Boot应用程序的声明式REST客户端。Feign通过可插拔的注解支持，包括Feign注解和JAX-RS注解，使编写Web服务客户端更加容易。  
此外，Spring Cloud还增加了对Spring MVC注解的支持，并支持使用与Spring Web相同的HttpMessageConverters。  
使用Feign的一个好处是，除了一个接口定义外，我们不需要写任何调用服务的代码。

### 集成步骤
1. 添加依赖
	````
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-openfeign</artifactId>
	</dependency>
	````
2. 接下来，我们需要将 `@EnableFeignClients` 添加到我们的主类中。有了这个注解，我们就可以对那些声明自己是Feign客户的接口进行组件扫描。
	```
	@SpringBootApplication
	@EnableFeignClients
	public class ExampleApplication {

		public static void main(String[] args) {
			SpringApplication.run(ExampleApplication.class, args);
		}
	}
	```
3. 然后我们使用 `@FeignClient` 注解来声明一个Feign客户端。
	```
	@FeignClient(value = "jplaceholder", url = "https://jsonplaceholder.typicode.com/")
	public interface JSONPlaceHolderClient {

		@RequestMapping(method = RequestMethod.GET, value = "/posts")
		List<Post> getPosts();

		@RequestMapping(method = RequestMethod.GET, value = "/posts/{postId}", produces = "application/json")
		Post getPostById(@PathVariable("postId") Long postId);
	}
	```
	在 `@FeignClient` 注解中传递的value参数是一个强制性的、任意的客户端名称，而通过url参数，我们指定了API的基本URL。
4. 可以在其他Bean 中注入声明的 Feign 客户端了。

### 配置
理解每个Feign客户端是由一组可定制的组件组成的，这一点非常重要。
Spring Cloud使用 `FeignClientsConfiguration` 类为每个命名的客户端按需创建一个默认配置选项。主要包括下述配置项：
+ Decoder – `ResponseEntityDecoder` ，它包装了 `SpringDecoder` ，用于解码 Response 响应。
+ Encoder – `SpringEncoder`， 用于编码请求。
+ Logger – 日志logger ,`Slf4jLogger` 是默认的日志。
+ Contract – `SpringMvcContract`, 它提供了注释处理
+ Feign-Builder – `HystrixFeign.Builder`，是用来创建 Feign 客户端组件的
+ Client – `LoadBalancerFeignClient` 或者默认的 Feign client

#### Java 代码配置
如果我们想定制这些Bean中的一个或多个，我们可以通过创建一个配置类来覆盖它们，然后将其添加到 FeignClient 注解的 configuration 属性中。
```
@FeignClient(value = "jplaceholder",
  url = "https://jsonplaceholder.typicode.com/",
  configuration = ClientConfiguration.class)

public class ClientConfiguration {
    @Bean
    public OkHttpClient client() {
        return new OkHttpClient();
    }
}
```

#### 配置文件配置
也可以使用配置文件来配置Feign客户端，而不是使用配置类，如这个application.yaml例子：
```
feign:
  client:
    config:
      default:
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: basic
```
通过这个配置，我们为应用程序中的每个声明的客户设置连接/读超时为5秒，日志级别为基本。

可以创建以 default 为客户端名称的配置来配置所有的 `@FeignClient` 对象，我们也可以为一个声明的特定 feign 客户端名称创建配置，如下配置专门配置名为 `jplaceholder` 的客户端：
```
feign:
  client:
    config:
      jplaceholder:
```
如果我们同时拥有 java 配置和配置文件配置，配置文件的属性将覆盖 java 配置的值。

#### Interceptors
如果我们需要在请求之前对request 进行特定的操作，可以使用 RequestInterceptor。
1. RequestInterceptor
	```
	@Bean
	public RequestInterceptor requestInterceptor() {
	return requestTemplate -> {
			requestTemplate.header("user", username);
			requestTemplate.header("password", password);
			requestTemplate.header("Accept", ContentType.APPLICATION_JSON.getMimeType());
		};
	}
	```
2. BasicAuthRequestInterceptor
   ```
    @Bean
	public BasicAuthRequestInterceptor basicAuthRequestInterceptor() {
		return new BasicAuthRequestInterceptor("username", "password");
	}
   ```




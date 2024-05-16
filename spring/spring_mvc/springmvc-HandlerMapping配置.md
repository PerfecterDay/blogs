# Spring-MVC 请求映射 HandlerMapping 配置
{docsify-updated}

`HandlerMapping` 就是将一个请求映射到一个处理请求的执行链 `HandlerExecutionChain` 。
```java
public interface HandlerMapping {
	@Nullable
	HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;
}
```

SpringMVC 提供了多种方式来配置 `HandlerMapping` 。

## @Controller和@RestController
将一个类标记为 Controller bean，这样 Spring MVC 的 RequestMapping 就会扫描到这个 bean 。

## 注册 handlerMapping 请求映射

### @RequestMapping
可以使 `@RequestMapping` 注解将请求映射到控制器方法。它有多种属性，可通过URL、HTTP方法、请求参数、头和媒体类型进行匹配。可以在类级别使用它来表达共享映射，也可以在方法级别使用它来缩小特定端点映射的范围。
RequestMapping 有很多属性可以指定匹配的细节：
+ `consumes` ： 匹配`Content-Type`是指定值的请求 ，`!text/plain` 指除 `text/plain` 以外的任何内容类型。
+ `params` : 匹配指定 `parameter=value` 的请求
+ `headers` : 匹配指定 `myHeader=myValue` 的请求,请注意，在 curl 语法中，头关键字和头值之间用冒号隔开，这与 HTTP 规范相同，而在 Spring 中则使用等号。
+ `produces` : 匹配特定的 `Accept` 请求头， 并且处理器会返回特定 `Accept` 指定的的响应媒体类型


可以在类级别使用 `consumes` 属性。在类级别使用时，方法级别的 `consumes` 属性会覆盖。
```
@PostMapping(path = "/pets", consumes = "application/json", ) 
```

`@RequestMapping` 方法可以使用 URL 模式进行映射。有两种选择：
1. `PathPattern` - 与URL路径匹配的预解析模式，也作为PathContainer进行预解析。该解决方案专为网络使用而设计，可有效处理编码和路径参数，并能高效匹配。
2. `AntPathMatcher` - 将字符串模式与字符串路径相匹配。这是最初的解决方案，也用于Spring配置，以选择classpath、文件系统和其他位置上的资源。它的效率较低，而且字符串路径输入对于有效处理编码和URL的其他问题是一个挑战。


### 特定方法的
`@RequestMapping` 也有 HTTP 方法特定的快捷方式变体：
+ `@GetMapping`
+ `@PostMapping`
+ `@PutMapping`
+ `@DeleteMapping`
+ `@PatchMapping`

示例：
+ "/resources/ima?e.png"： - 匹配路径段中的一个字符
+ "/resources/*.png"： - 匹配路径段中的零个或多个字符
+ "/resources/**"： - 匹配多个路径段
+ "/projects/{project}/versions"： - 匹配路径段并将其作为变量捕获
+ "/projects/{project:[a-z]}/versions"+ ： - 使用 regex 匹配并捕获变量

### 显示的注册

```java
@Configuration
public class MyConfig {
	@Autowired
	public void setHandlerMapping(RequestMappingHandlerMapping mapping, UserHandler handler)
			throws NoSuchMethodException {
		RequestMappingInfo info = RequestMappingInfo
				.paths("/user/{id}").methods(RequestMethod.GET).build();
		Method method = UserHandler.class.getMethod("getUser", Long.class);
		mapping.registerMapping(info, handler, method);
	}
}
```

## 响应相关的注解

### @ResponseBody
可以在方法上使用 `@ResponseBody` 注解，通过 `HttpMessageConverter` 将返回序列化为响应体:
```
@GetMapping("/accounts/{id}")
@ResponseBody
public Account handle() {
}
```
`@ResponseBody` 在类级别也被支持，在这种情况下，它被所有控制器方法继承。这就是 `@RestController` 的效果，它只不过是一个标有 `@Controller` 和 `@ResponseBody` 的元注解。

### ResponseEntity
`ResponseEntity` 类似于 `@ResponseBody` ，但带有状态和头信息。
```
@GetMapping("/something")
public ResponseEntity<String> handle() {
	String body = ... ;
	String etag = ... ;
	return ResponseEntity.ok().eTag(etag).body(body);
}
```

Spring MVC支持异步生成ResponseEntity，或为主体使用单值和多值反应类型。这允许以下类型的异步响应：
+ `ResponseEntity<Mono<T>>` 或 `ResponseEntity<Flux<T>>` 可立即知道响应状态和标头，而正文则在稍后异步提供。如果正文由 0...1 个值组成，请使用 Mono；如果可以产生多个值，请使用 Flux。
+ `Mono<ResponseEntity<T>>` 可在稍后异步提供所有三项内容--响应状态、头信息和正文。这允许响应状态和头信息根据异步请求处理的结果而变化。
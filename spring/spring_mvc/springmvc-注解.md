# Spring-MVC 相关的注解
{docsify-updated}

- [Spring-MVC 相关的注解](#spring-mvc-相关的注解)
	- [RequestMapping 匹配请求相关的注解](#requestmapping-匹配请求相关的注解)
		- [@Controller/@RestController](#controllerrestcontroller)
		- [@RequestMapping](#requestmapping)
		- [特定方法的](#特定方法的)
	- [获取请求中作为方法参数的注解](#获取请求中作为方法参数的注解)
		- [获取矩阵变量 @PathVariable/@MatrixVariable](#获取矩阵变量-pathvariablematrixvariable)
		- [获取请求参数 @RequestParam](#获取请求参数-requestparam)
		- [获取请求头参数 @RequestHeader](#获取请求头参数-requestheader)
		- [获取Cookie值 @CookieValue](#获取cookie值-cookievalue)
		- [@SessionAttributes 存储会话属性](#sessionattributes-存储会话属性)
		- [@SessionAttribute 获取会话属性](#sessionattribute-获取会话属性)
		- [@RequestBody 获取请求体](#requestbody-获取请求体)
		- [HttpEntity](#httpentity)
	- [响应相关的注解](#响应相关的注解)
		- [@ResponseBody](#responsebody)
		- [ResponseEntity](#responseentity)


## RequestMapping 匹配请求相关的注解

### @Controller/@RestController
将一个类标记为 Controller bean，这样 Spring MVC 的 RequestMapping 就会扫描到这个 bean 。


### @RequestMapping
可以使 `@RequestMapping` 注解将请求映射到控制器方法。它有多种属性，可通过URL、HTTP方法、请求参数、头和媒体类型进行匹配。可以在类级别使用它来表达共享映射，也可以在方法级别使用它来缩小特定端点映射的范围。
RequestMapping 有很多属性可以指定匹配的细节：
`consumes` ： 匹配指定请求的特定的 `Content-Type` ， `!text/plain` 指除 `text/plain` 以外的任何内容类型。
`params` : 匹配指定 `parameter=value` 的请求
`headers` : 匹配指定 `myHeader=myValue` 的请求

```
@PostMapping(path = "/pets", consumes = "application/json", ) 
```

显示的注册：

```
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

@RequestMapping 方法可以使用 URL 模式进行映射。有两种选择：
1. PathPattern - 与URL路径匹配的预解析模式，也作为PathContainer进行预解析。该解决方案专为网络使用而设计，可有效处理编码和路径参数，并能高效匹配。
2. AntPathMatcher - 将字符串模式与字符串路径相匹配。这是最初的解决方案，也用于Spring配置，以选择classpath、文件系统和其他位置上的资源。它的效率较低，而且字符串路径输入对于有效处理编码和URL的其他问题是一个挑战。


### 特定方法的
`@RequestMapping` 也有 HTTP 方法特定的快捷方式变体：
+ `@GetMapping`
+ `@PostMapping`
+ `@PutMapping`
+ `@DeleteMapping`
+ `@PatchMapping`

## 获取请求中作为方法参数的注解

### 获取矩阵变量 @PathVariable/@MatrixVariable
[RFC 3986](https://tools.ietf.org/html/rfc3986#section-3.3) 讨论了路径段中的名-值对。在Spring MVC中，将其称为 "矩阵变量"，但它们也可以被称为URI路径参数。  

矩阵变量可以出现在任意路径段中，每个变量之间用分号隔开，多个值之间用逗号隔开（例如，/cars;color=red,green;year=2012）。也可以通过重复变量名指定多个值（例如，color=red;color=green;color=blue）。

如果预期 URL 包含矩阵变量，控制器方法的请求映射必须使用 URI 变量来屏蔽变量内容，并确保请求能够成功匹配，而与矩阵变量的顺序和存在无关。下面的示例使用了矩阵变量：
```
// GET /owners/42;q=11/pets/21;q=22/2
@GetMapping("/owners/{ownerId}/pets/{petId}/{id}")
public void findPet(
		@MatrixVariable(name="q", pathVar="ownerId") int q1,
		@MatrixVariable(name="q", pathVar="petId") int q2,
		@PathVariable String id) {
	// q1 == 11
	// q2 == 22
	// id == 2
}
```

### 获取请求参数 @RequestParam
您可以使用 `@RequestParam` 注解将 Servlet 请求参数（即查询参数或表单数据）绑定到控制器中的方法参数。
```
@GetMapping
public String setupForm(@RequestParam("petId") int petId, Model model) { 
	Pet pet = this.clinic.loadPet(petId);
	model.addAttribute("pet", pet);
	return "petForm";
}
```

### 获取请求头参数 @RequestHeader
`@RequestHeader` 可以获取请求头中的参数。
```
@GetMapping("/demo")
public void handle(
		@RequestHeader("Accept-Encoding") String encoding,
		@RequestHeader("Keep-Alive") long keepAlive) {
	//...
}
```
当 `@RequestHeader` 注解被用于 `Map<String,String>、MultiValueMap<String,String> 或 HttpHeaders` 参数时，该 `Map` 将被填充所有头值。


### 获取Cookie值 @CookieValue
可以使用 `@CookieValue` 注解将 HTTP cookie 的值绑定到控制器中的方法参数。
```
@GetMapping("/demo")
public void handle(@CookieValue("JSESSIONID") String cookie) {
	//...
}
```

### @SessionAttributes 存储会话属性
`@SessionAttributes` 用于在请求之间的会话Session中存储模型属性。它是一个类级别的注解，用于声明特定控制器使用的会话属性。这通常列出了模型属性的名称或模型属性的类型，这些属性应透明地存储在会话中，供后续请求访问。  
在第一次请求中，当一个名称为 pet 的模型属性被添加到模型中时，它被自动提升并保存在 HTTP Servlet 会话中。如下例所示，在另一个控制器方法使用 SessionStatus 方法参数清除存储之前，该属性一直保留在会话中：
```
@Controller
@SessionAttributes("pet")
public class EditPetForm {
	@PostMapping("/pets/{id}")
	public String handle(Pet pet, BindingResult errors, SessionStatus status) {
		if (errors.hasErrors) {
			// ...
		}
		status.setComplete();
		// ...
	}
}
```

### @SessionAttribute 获取会话属性
如果您需要访问可能存在也可能不存在的已有会话属性，您可以在方法参数上使用 `@SessionAttribute` 注解，如下例所示：
```
@RequestMapping("/")
public String handle(@SessionAttribute User user) {
	// ...
}
```

### @RequestBody 获取请求体
可以使用 `@RequestBody` 注解通过 `HttpMessageConverter` 将请求 body 读取并反序列化为对象。下面的示例使用了 `@RequestBody` 参数：
```
@PostMapping("/accounts")
public void handle(@Valid @RequestBody Account account, BindingResult result) {
	// ...
}
```

### HttpEntity
`HttpEntity` 与使用 `@RequestBody` 大致相同，但它基于一个容器对象，该容器对象暴露了请求头和请求体:
```
@PostMapping("/accounts")
public void handle(HttpEntity<Account> entity) {
	// ...
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















# Spring-REST Clients
{docsify-updated}

Spring Framework 为调用 REST 端点提供了以下选项：
+ `RestClient` — 同步调用的 fluent API
+ `WebClient` — 非阻塞响应式 fluent API
+ `RestTemplate` — 采用模板方法 API 的同步客户端（现已弃用，推荐使用 `RestClient` ）
+ `HTTP Service Clients` — 基于生成代理的注解接口


## RestClient
`RestClient` 是一个同步 HTTP 客户端，提供 fluent API 用于执行请求。它作为 HTTP 库的抽象层，负责将 HTTP 请求和响应内容转换为更高层次的 Java 对象，并反之亦然。

### 创建 RestClient
`RestClient` 提供了静态创建快捷方法。它还暴露了一个 `builder()` 方法，支持更多选项：
+ 选择要使用的底层HTTP库
+ 配置消息转换器
+ 设置基础URL
+ 设置默认请求头、Cookie、路径变量及API版本
+ 配置 `ApiVersionInserter`
+ 注册拦截器
+ 注册请求初始化器

一旦创建， `RestClient` 可以被多线程安全的使用。
```
RestClient defaultClient = RestClient.create();

RestClient customClient = RestClient.builder()
	.requestFactory(new HttpComponentsClientHttpRequestFactory()) //设置底层使用的 http 库
	.messageConverters(converters -> converters.add(new MyCustomMessageConverter()))
	.baseUrl("https://example.com")
	.defaultUriVariables(Map.of("variable", "foo"))
	.defaultHeader("My-Header", "Foo")
	.defaultCookie("My-Cookie", "Bar")
	.defaultVersion("1.2")
	.apiVersionInserter(ApiVersionInserter.fromHeader("API-Version").build())
	.requestInterceptor(myCustomInterceptor)
	.requestInitializer(myCustomInitializer)
	.build();
```

#### 选择并配置底层的 http 库
为执行HTTP请求， `RestClient` 采用需要依赖HTTP客户端库。这些库通过 `ClientRequestFactory` 接口进行适配，提供多种实现方案：
+ `JdkClientHttpRequestFactory` ：使用Java JDK的 `HttpClient`
+ `HttpComponentsClientHttpRequestFactory` ：适用于Apache HTTP Components 的 `HttpClient`
+ `JettyClientHttpRequestFactory` ：适用于Jetty的 `HttpClient`
+ `ReactorNettyClientRequestFactory` ：用于 Reactor Netty 的 `HttpClient`
+ `SimpleClientHttpRequestFactory` ：作为简单默认实现

若 RestClient 构建时未指定请求工厂，则优先使用类路径中存在的 Apache 或 Jetty `HttpClient` 。若未找到，且加载了 java.net.http 模块，则使用 Java 的 `HttpClient` 。最后才会采用简单默认实现。

如果需要对底层http client 库进行一些配置，如连接超时时间、连接池大小、read-write timeout 等底层设置时，可以构造一个配置好的 client 对象，并初始化到上述对饮的 `ClientRequestFactory` 实现中，如：
```
HttpClient httpClient = HttpClient.newBuilder()
                .connectTimeout(Duration.ofSeconds(5))   // 建连超时
                .followRedirects(HttpClient.Redirect.NORMAL)
                .version(HttpClient.Version.HTTP_2)
                .build();

RestClient restClient = RestClient.builder()
		.baseUrl("https://671d110e-33d7-449f-b7f7-7387077781df.mock.pstmn.io")
		.requestFactory(new JdkClientHttpRequestFactory(httpClient))
		.build();

ResponseEntity<ApiResponse> result = restClient.post()
		.uri("https://671d110e-33d7-449f-b7f7-7387077781df.mock.pstmn.io/am5/rest/OATH/create-assign-and-encrypt")
		.contentType(MediaType.APPLICATION_JSON)
		.header("x-api-key", "xxxx")
		.header("x-mock-response-name", "aaa")
		.body("{}")
		.retrieve()
		.toEntity(ApiResponse.class);

System.out.println("Response status: " + result.getStatusCode());
System.out.println("Response headers: " + result.getHeaders());
System.out.println("Contents: " + result.getBody());
```

当你不关心返回值body时，可以使用 `toBodilessEntity()` 方法。

#### 异常处理
```
String result = restClient.get() 
	.uri("https://example.com/this-url-does-not-exist") 
	.retrieve()
	.onStatus(HttpStatusCode::is4xxClientError, (request, response) -> { 
		throw new MyCustomRuntimeException(response.getStatusCode(), response.getHeaders()); 
	})
	.body(String.class);
```

#### Exchange 方法
对于更高级的场景， `RestClient` 通过 `exchange()` 方法提供对底层 HTTP 请求和响应的访问，该方法可替代 `retrieve()` 使用。使用 `exchange()` 时不能使用 `onStatus()` 的错误处理方法 ，因为 `exchange` 函数本身已提供对完整响应的访问权限，允许您执行任何必要的错误处理。
```
Pet result = restClient.get()
	.uri("https://petclinic.example.com/pets/{id}", id)
	.accept(APPLICATION_JSON)
	.exchange((request, response) -> {
		if (response.getStatusCode().is4xxClientError()) {
			throw new MyCustomRuntimeException(response.getStatusCode(), response.getHeaders());
		}
		else {
			Pet pet = convertResponse(response);
			return pet;
		}
	});
```

#### multipart
```
MultiValueMap<String, Object> parts = new LinkedMultiValueMap<>();

parts.add("fieldPart", "fieldValue");
parts.add("filePart", new FileSystemResource("...logo.png"));
parts.add("jsonPart", new Person("Jason"));

HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_XML);
parts.add("xmlPart", new HttpEntity<>(myBean, headers));

// send using RestClient.post or RestTemplate.postForEntity
```

## WebClient

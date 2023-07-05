# Spring MVC 的注解、参数校验与异常处理
{docsify-updated}

- [Spring MVC 的注解、参数校验与异常处理](#spring-mvc-的注解参数校验与异常处理)
	- [常用注解](#常用注解)
		- [HTTP请求相关的注解](#http请求相关的注解)
		- [HTTP 相应相关的注解](#http-相应相关的注解)
		- [其他相关注解](#其他相关注解)
	- [统一异常处理](#统一异常处理)
		- [使用 @ExceptionHandler 注解](#使用-exceptionhandler-注解)
		- [实现 HandlerExceptionResolver 接口并注册到 bean 容器](#实现-handlerexceptionresolver-接口并注册到-bean-容器)
		- [使用 @ControllerAdvice+ @ExceptionHandler 注解](#使用-controlleradvice-exceptionhandler-注解)



## 常用注解
### HTTP请求相关的注解
1. `@RequestMapping`   

	`@RequestMapping` 通常用在标注了 `@Controller` 的类或方法上，用来匹配处理的 URL ，该注解可以有以下属性:
	1. `path`: 和 `name` 、 `value` 都是同样的，用来指定这个方法或类匹配处理哪个方法  
	2. `method`: 匹配 HTTP 请求方法  
	3. `params`: 可以根据指定的参数是否出现或者等于指定值来确定是否匹配这个URL
	4. `headers`: 可以根据指定的HTTP 头是否出现或者等于指定值来确定是否匹配这个URL
	5. `consumes`: 指定该方法可以处理的HTTP 的 media type
	6. `produces`: 指定该方法生成的HTTP响应的 media type  
	该注解如果用在类上，那么该类中的所有方法会“继承”它的属性，如果方法也加了该注解，那么两个注解的属性通常会叠加而不是覆盖（如果类上指定GET，方法指定POST则会覆盖）。
	类似的 `@GetMapping` , `@PostMapping` , `@PutMapping` , `@DeleteMapping` 和 `@PatchMapping` 等同于 `@RequestMapping` 用于 method 指定的匹配的 HTTP method。

2. `@RequestBody`  
	将HTTP请求的请求体映射到一个对象。
	```
	@PostMapping("/save")
	void saveVehicle(@RequestBody Vehicle vehicle) {
		// ...
	}
	```
3. `@PathVariable`  
	将URL中的路径参数绑定到一个方法参数上。路径参数是 Spring 实现的，官方名字就叫做 URI template variable。
	它有 name 和 required（true/false） 两个属性。
	```
	如果路径是 /1234
	@RequestMapping("/{id}")
	Vehicle getVehicle(@PathVariable("id") long id) {
		// ... id=1234
	}
	```
4. `@RequestParam`  
	该注解可以用来绑定HTTP的请求参数。可以使用 defaultValue 参数指定默认值，这样请求参数自动变成可选的。
	```
	@RequestMapping("/buy")
	Car buyCar(@RequestParam(defaultValue = "5") int seatCount) {
		// ...
	}
	```
	与这个注解类似的还有 `@CookieValue` 和 `@RequestHeader`，分别用来绑定 Cookie 和请求头中的参数。

### HTTP 相应相关的注解
1. `@ResponseBody`  
该注解会将方法的响应值作为HTTP 响应体发送给客户端。可以用在类上，那么类中所有方法的返回值将直接作为响应体返回。
2. `@ResponseStatus`  
指定返回的HTTP code  
`@ResponseStatus(value = HttpStatus.FORBIDDEN, reason="To show an example of a custom message")`

### 其他相关注解
1. `@Controller`:指定一个处理 HTTP 请求的类
2. `@RestControlle`: 相当于 `@Controller` + `@ResponseBody`
3. `CrossOrigin`: 允许跨域

## 统一异常处理
Spring 统一异常处理有 3 种方式，分别为：

1. 使用 @ ExceptionHandler 注解
2. 实现 HandlerExceptionResolver 接口
3. 使用 @controlleradvice 注解

### 使用 @ExceptionHandler 注解
使用该注解有一个不好的地方就是：进行异常处理的方法必须与出错的方法在同一个Controller里面。使用如下：
```
@Controller      
public class GlobalController {               
 
   /**    
     * 用于处理异常的    
     * @return    
     */      
    @ExceptionHandler({MyException.class})       
    public String exception(MyException e) {       
        System.out.println(e.getMessage());       
        e.printStackTrace();       
        return "exception";       
    }       
 
    @RequestMapping("test")       
    public void test() {       
        throw new MyException("出错了！");       
    }                    
```
这种方式最大的缺陷就是不能全局控制异常。每个类都要写一遍。但是这种方法对普通的 Controller 和 RestController 都可以使用。

### 实现 HandlerExceptionResolver 接口并注册到 bean 容器
这种方式可以进行全局的异常控制，但是只对普通的 Controller 有效，对于 RestController 无效。
```
@Component
public class UnifiedExceptionResolver implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {

        e.printStackTrace();
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.addObject("exception",e);
        return modelAndView;
    }
}
```
同时使用 HandlerExceptionResolver 和 @ExceptionHandler 注解时，@ExceptionHandler 会覆盖 HandlerExceptionResolver 。

### 使用 @ControllerAdvice+ @ExceptionHandler 注解
上文说到 @ ExceptionHandler 需要进行异常处理的方法必须与出错的方法在同一个Controller里面。那么当代码加入了 @ControllerAdvice，则不需要必须在同一个 controller 中了。这也是 Spring 3.2 带来的新特性。从名字上可以看出大体意思是控制器增强。 也就是说，@controlleradvice + @ ExceptionHandler 也可以实现全局的异常捕捉，请确保此WebExceptionHandle 类能被扫描到并装载进 Spring 容器中。
```
@ControllerAdvice
@ResponseBody
public class WebExceptionHandler {
    private static Logger logger = LoggerFactory.getLogger(WebExceptionHandle.class);
    /**
     * 400 - Bad Request
     */
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(HttpMessageNotReadableException.class)
    public ServiceResponse handleHttpMessageNotReadableException(HttpMessageNotReadableException e) {
        logger.error("参数解析失败", e);
        return ServiceResponseHandle.failed("could_not_read_json");
    }
    
     /**
     * 405 - Method Not Allowed
     */
    @ResponseStatus(HttpStatus.METHOD_NOT_ALLOWED)
    @ExceptionHandler()
    public ServiceResponse handleHttpRequestMethodNotSupportedException(HttpRequestMethodNotSupportedException e) {
        logger.error("不支持当前请求方法", e);
        return ServiceResponseHandle.failed("request_method_not_supported");
    }


    /**
     * 500 - Internal Server Error
     */
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler(Exception.class)
    public ServiceResponse handleException(Exception e) {
        if (e instanceof BusinessException){
            return ServiceResponseHandle.failed("BUSINESS_ERROR", e.getMessage());
        }
        
        logger.error("服务运行异常", e);
        e.printStackTrace();
        return ServiceResponseHandle.failed("server_error");
    }
}
```
如果 @ExceptionHandler 注解中未声明要处理的异常类型，则默认为参数列表中的异常类型。参见上面的 405 异常处理。

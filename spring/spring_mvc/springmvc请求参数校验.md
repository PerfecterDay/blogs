# Spring MVC 请求参数校验及统一异常处理
{docsify-updated}

## 请求参数验证

### JSR-303
JSR-303是Java为Bean数据合法性校验提供的标准框架，它定义了一套可标注在成员变量，属性方法上的校验注解。
Hibernate Validation提供了这套标准的实现，在我们引入Spring boot starter validation的时候，默认会引入Hibernate Validation。

### Spring 使用  validation 的步骤
1. 为业务对象bean 添加相应的验证注解
```
@Data
public class User {
 // 名字不允许为空，并且名字的长度在2位到30位之间
 // 如果名字的长度校验不通过，那么提示错误信息
 @NotNull
 @Size(min=2, max=30,message = "请检查名字的长度是否有问题")
 private String name;

 // 不允许为空，并且年龄的最小值为18
 @NotNull
 @Min(18)
 private Integer age;
}
```
2. 控制器内对验证对象前加上 @Valid 注解，验证结果会返回到 BindingResult 对象中
```
// 1. 要校验的参数前，加上@Valid注解
 // 2. 紧随其后的，跟上一个BindingResult来存储校验信息
 @RequestMapping("/test1")
 public Object test1(@Valid User user,BindingResult bindingResult) {
 //如果检验出了问题，就返回错误信息
 // 这里我们返回的是全部的错误信息，实际中可根据bindingResult的方法根据需要返回自定义的信息。
 // 通常的解决方案为：JSR-303 + 全局ExceptionHandler
 if (bindingResult.hasErrors()){
  return bindingResult.getAllErrors();
 }
 return "OK";
 } 
 ```

### 常见的校验注解
JSR-303 提供的标准注解：
+ @Null 被注释的元素必须为 null
+ @NotNull 被注释的元素必须不为 null
+ @AssertTrue 被注释的元素必须为 true
+ @AssertFalse 被注释的元素必须为 false
+ @Min(value) 被注释的元素必须是一个数字，其值必须大于等于指定的最小值
+ @Max(value) 被注释的元素必须是一个数字，其值必须小于等于指定的最大值
+ @DecimalMin(value) 被注释的元素必须是一个数字，其值必须大于等于指定的最小值
+ @DecimalMax(value) 被注释的元素必须是一个数字，其值必须小于等于指定的最大值
+ @Size(max=, min=) 被注释的元素的大小必须在指定的范围内
+ @Digits (integer, fraction) 被注释的元素必须是一个数字，其值必须在可接受的范围内
+ @Past 被注释的元素必须是一个过去的日期
+ @Future 被注释的元素必须是一个将来的日期
+ @Pattern(regex=,flag=) 被注释的元素必须符合指定的正则表达式

Hibernate Validator提供的校验注解：
+ @NotBlank(message =) 验证字符串非null，且长度必须大于0
+ @Email 被注释的元素必须是电子邮箱地址
+ @Length(min=,max=) 被注释的字符串的大小必须在指定的范围内
+ @NotEmpty 被注释的字符串的必须非空
+ @Range(min=,max=,message=) 被注释的元素必须在合适的范围内

### 自定义校验注解
有时候，第三方库中并没有我们想要的校验类型，好在系统提供了很好的扩展能力，我们可以自定义检验。  
比如，我们想校验用户的手机格式，写手机号码校验器

1、编写校验注解
```
// 我们可以直接拷贝系统内的注解如@Min，复制到我们新的注解中，然后根据需要修改。
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER})
@Retention(RUNTIME)
@Documented
//注解的实现类。
@Constraint(validatedBy = {IsMobileValidator.class})
public @interface IsMobile {
 //校验错误的默认信息
 String message() default "手机号码格式有问题";
 //是否强制校验
 boolean isRequired() default false;
 Class<?>[] groups() default {};
 Class<? extends Payload>[] payload() default {};
}
```

2、编写具体的实现类
我们知道注解只是一个标记，真正的逻辑还要在特定的类中实现，上一步的注解指定了实现校验功能的类为 `IsMobileValidator` 。
```
// 自定义注解一定要实现ConstraintValidator接口奥，里面的两个参数
// 第一个为 具体要校验的注解
// 第二个为 校验的参数类型
public class IsMobileValidator implements ConstraintValidator<IsMobile, String> {

 private boolean required = false;

 private static final Pattern mobile_pattern = Pattern.compile("1\\d{10}");
 //工具方法，判断是否是手机号
 public static boolean isMobile(String src) {
  if (StringUtils.isEmpty(src)) {
   return false;
  }
  Matcher m = mobile_pattern.matcher(src);
  return m.matches();
 }

 @Override
 public void initialize(IsMobile constraintAnnotation) {
  required = constraintAnnotation.isRequired();
 }

 @Override
 public boolean isValid(String phone, ConstraintValidatorContext constraintValidatorContext) {
  //是否为手机号的实现
  if (required) {
   return isMobile(phone);
  } else {
   if (StringUtils.isEmpty(phone)) {
    return true;
   } else {
    return isMobile(phone);
   }
  }
 }
}
```

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

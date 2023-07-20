## SpringMvc 的处理方法参数绑定与校验
{docsify-updated}

- [SpringMvc 的处理方法参数绑定与校验](#springmvc-的处理方法参数绑定与校验)
	- [参数绑定- HandlerMethodArgumentResolver 及 HttpMessageConverter](#参数绑定--handlermethodargumentresolver-及-httpmessageconverter)
	- [数据绑定流程剖析](#数据绑定流程剖析)
- [请求参数验证](#请求参数验证)
	- [Spring 参数校验的原理](#spring-参数校验的原理)
	- [JSR-303](#jsr-303)
	- [Spring 使用  validation 的步骤](#spring-使用--validation-的步骤)
	- [常见的校验注解](#常见的校验注解)
	- [自定义校验注解](#自定义校验注解)


Spring 会根据请求方法签名的不同，将请求消息中的信息 以一定的方式转换并绑定到请求方法的入参中。当请求消息到达真正需要调用的方法时(如指定的业务方法)，Spring MVC 还有很多工作要做，包括数据转换、数据格式化及数据校验等。

### 参数绑定- HandlerMethodArgumentResolver 及 HttpMessageConverter
常见的参数解析器： `PathVariableMethodArgumentResolver` , `RequestResponseBodyMethodProcessor`, `RequestHeaderMethodArgumentResolver`
会用到很多 `HttpMessageConverter`

```
RequestMappingHandlerAdapter.invokeHandlerMethod()->ServletInvocableHandlerMethod.invokeAndHandle()
->InvocableHandlerMethod.invokeForRequest()->HandlerMethodArgumentResolverComposite.resolveArgument()
->PathVariableMethodArgumentResolver.resolveArgument() 
```
核心代码：
```
parameter = parameter.nestedIfOptional();
Object arg = readWithMessageConverters(webRequest, parameter, parameter.getNestedGenericParameterType());
String name = Conventions.getVariableNameForParameter(parameter);

if (binderFactory != null) {
	WebDataBinder binder = binderFactory.createBinder(webRequest, arg, name);
	if (arg != null) {
		validateIfApplicable(binder, parameter);
		if (binder.getBindingResult().hasErrors() && isBindExceptionRequired(binder, parameter)) {
			throw new MethodArgumentNotValidException(parameter, binder.getBindingResult());
		}
	}
	if (mavContainer != null) {
		mavContainer.addAttribute(BindingResult.MODEL_KEY_PREFIX + name, binder.getBindingResult());
	}
}
```

HandlerMethodArgumentResolver（RequestResponseBodyMethodProcessor） -> HttpMessageConverter（MappingJackson2HttpMessageConverter）
-> WebDataBinder


### 数据绑定流程剖析
Spring MvC通过反射机制对目标处理方法的签名进行分析，将请求消息绑定到处理方法的入参中。数据鄉定的核心部件是DataBinder，其运行机制描述如下所示：
<center><img src="pics/databind.jpg" width="40%"></center>

Spring MVC 主框架将 ServletRequest 对象及处理方法的入参对象实例传递给 DataBinder, DataBinder 首先调用装配在SpringWeb 上下文中的 ConversionService 组件进行数据类型转换、数据格式化等工作，将 ServletRequest 中的消息填充到入参对象中， 然后调用 Validator 组件对己经鄉定了请求消息数据的入参对象进行数据合法性校验，最终生成数据绑定结果 BindingResult 对象。 BindingResult 包含了已完成数据绑定的入参 对象，还包含相应的校验错误对象。Spring MVC 抽取 BindingResult 中的入参对象及校验错误对象，将它们赋给处理方法的相应入参。


https://www.baeldung.com/spring-data-redis-pub-sub

https://github.com/Homebrew/discussions/discussions/2530
https://www.baeldung.com/spring-retry

## 请求参数验证

### Spring 参数校验的原理



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
+ `@Null` 被注释的元素必须为 null
+ `@NotNull` 被注释的元素必须不为 null
+ `@AssertTrue` 被注释的元素必须为 true
+ `@AssertFalse` 被注释的元素必须为 false
+ `@Min(value)` 被注释的元素必须是一个数字，其值必须大于等于指定的最小值
+ `@Max(value)` 被注释的元素必须是一个数字，其值必须小于等于指定的最大值
+ `@DecimalMin(value)` 被注释的元素必须是一个数字，其值必须大于等于指定的最小值
+ `@DecimalMax(value)` 被注释的元素必须是一个数字，其值必须小于等于指定的最大值
+ `@Size(max=, min=)` 被注释的元素的大小必须在指定的范围内
+ `@Digits (integer, fraction)` 被注释的元素必须是一个数字，其值必须在可接受的范围内
+ `@Past` 被注释的元素必须是一个过去的日期
+ `@Future` 被注释的元素必须是一个将来的日期
+ `@Pattern(regex=,flag=)` 被注释的元素必须符合指定的正则表达式

Hibernate Validator提供的校验注解：
+ `@NotBlank(message =)` 验证字符串非null，且长度必须大于0
+ `@Email` 被注释的元素必须是电子邮箱地址
+ `@Length(min=,max=)` 被注释的字符串的大小必须在指定的范围内
+ `@NotEmpty` 被注释的字符串的必须非空
+ `@Range(min=,max=,message=)` 被注释的元素必须在合适的范围内

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
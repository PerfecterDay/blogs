# Validator
{docsify-updated}

> https://docs.spring.io/spring-framework/reference/core/validation/validator.html


Spring 提供了一个 `Validator` 接口，可用于验证对象。在验证过程中， `Validator` 能够将验证失败的结果报告给 `Errors` 对象。

`Validator` 中包含两个方法：
+ `supports(Class)` : 确定这个 `Validator` 是否支持指定 `Class` 的验证
+ `validate(Object, org.springframework.validation.Errors)` : 验证一个对象，如果有验证错误，将错误信息通过 `Errors` 返回

实现 `Validator` 相当简单，尤其当你了解Spring框架提供的 `ValidationUtils` 辅助类时。以下示例实现了 `Person` 实例的 `Validator` ：
```
public class Person {

	private String name;
	private int age;

	// the usual getters and setters...
}

public class PersonValidator implements Validator {

	/**
	 * This Validator validates only Person instances
	 */
	public boolean supports(Class clazz) {
		return Person.class.equals(clazz);
	}

	public void validate(Object obj, Errors e) {
		ValidationUtils.rejectIfEmpty(e, "name", "name.empty");
		Person p = (Person) obj;
		if (p.getAge() < 0) {
			e.rejectValue("age", "negativevalue");
		} else if (p.getAge() > 110) {
			e.rejectValue("age", "too.darn.old");
		}
	}
}
```

虽然完全可以实现一个单一的 `Validator` 类来验证丰富对象中的每个嵌套对象，但更好的做法是将每个嵌套对象类的验证逻辑封装在其专属的 `Validator` 实现中。一个简单的"丰富"对象示例是 `Customer` 类，它由两个 `String` 属性（名字和姓氏）以及一个复杂的 `Address` 对象组成。由于 `Address` 对象可独立于 `Customer` 对象使用，因此已实现独立的 `AddressValidator` 。若希望 `CustomerValidator` 复用 `AddressValidator` 类中的逻辑而不依赖复制粘贴，可通过依赖注入或在 `CustomerValidator` 内部实例化 `AddressValidator` 实现，如下例所示：
```
public class CustomerValidator implements Validator {

	private final Validator addressValidator;

	public CustomerValidator(Validator addressValidator) {
		if (addressValidator == null) {
			throw new IllegalArgumentException("The supplied [Validator] is " +
				"required and must not be null.");
		}
		if (!addressValidator.supports(Address.class)) {
			throw new IllegalArgumentException("The supplied [Validator] must " +
				"support the validation of [Address] instances.");
		}
		this.addressValidator = addressValidator;
	}

	/**
	 * This Validator validates Customer instances, and any subclasses of Customer too
	 */
	public boolean supports(Class clazz) {
		return Customer.class.isAssignableFrom(clazz);
	}

	public void validate(Object target, Errors errors) {
		ValidationUtils.rejectIfEmptyOrWhitespace(errors, "firstName", "field.required");
		ValidationUtils.rejectIfEmptyOrWhitespace(errors, "surname", "field.required");
		Customer customer = (Customer) target;
		try {
			errors.pushNestedPath("address");
			ValidationUtils.invokeValidator(this.addressValidator, customer.getAddress(), errors);
		} finally {
			errors.popNestedPath();
		}
	}
}
```

 `Validator` 也可在本地调用，用于对给定对象进行即时验证，无需涉及绑定过程。从6.1版本起，此功能通过新增的 `Errors Validator.validateObject(Object)` 方法得到简化，该方法现已默认可用，返回 `Errors` 对象：通常调用 `hasErrors()` 方法，或使用新增的 `Errors.failOnError` 方法将错误摘要转换为异常（例如： `validator.validateObject(myObject).failOnError(IllegalArgumentException::new)` ）。
```
public interface Errors {
    ....
    default <T extends Throwable> void failOnError(Function<String, T> messageToException) throws T {
		if (hasErrors()) {
			throw messageToException.apply(toString());
		}
	}
    ...
}
```

## Jakarta Bean Validation
Bean Validation 通过 Java 应用程序的约束声明和元数据提供了一种通用的验证方法。要使用它，需要使用声明式验证约束注解到想要验证的字段上，然后由运行时执行校验。JSR提供了内置的约束，您也可以定义自己的自定义约束。

Jakarta Bean Validation 是Java为Bean数据合法性校验提供的标准框架，它定义了一套可标注在成员变量，属性方法上的校验注解。是一套JSR标准。  
Hibernate Validation提供了这套标准的实现，在我们引入Spring boot starter validation的时候，默认会引入Hibernate Validation。

下边是一个使用 Jakarta Bean Validation（Hibernate Validator实现）的示例：
```
@Data
class Person{
    @NotBlank
    @NotNull
    private String name;
    @Min(200)
    private int age;

    public static void main(String[] args) {
        ValidatorFactory validatorFactory = Validation.byProvider( HibernateValidator.class )
                .configure()
                .buildValidatorFactory();
        javax.validation.Validator validator = validatorFactory.getValidator();

        Person p = new Person();
        p.setAge(120);
        Set<ConstraintViolation<Person>> constraintViolations =
                validator.validate( p );
        System.out.println(constraintViolations.size());
    }
}
```
<center><img src="pics/hibernate-validator.jpg" alt=""></center>

### Hibernate Validator 的SPI 机制
<center><img src="pics/hibernate-validator-spi.png" alt=""></center>

### Spring 使用 validation 的步骤
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
	// 通常的解决方案为：Jakarta Bean Validation + 全局ExceptionHandler
	if (bindingResult.hasErrors()){
	return bindingResult.getAllErrors();
	}
	return "OK";
	} 
	```

### 常见的Jakarta校验注解
Jakarta Bean Validation 提供的标准注解：
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
message/groups/payload 这些成员必须要有。

2、编写具体的实现类

注解只是一个标记，真正的逻辑还要在特定的类中实现，上一步的注解指定了实现校验功能的类为 `IsMobileValidator` 。
```
// 自定义注解一定要实现ConstraintValidator接口奥，里面的两个参数
// 第一个为 具体要校验的注解
// 第二个为 校验的参数类型
public class IsMobileValidator implements ConstraintValidator<IsMobile, String> {

 @Autowired;
 private Foo aDependency;
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
与其他 Spring Bean 一样， `ConstraintValidator` 实现也可以将其依赖 bean 设置为 `@Autowired` 以注入。并且 `ConstraintValidator` 实现类**必须是 public 的**。

### Spring 对 Jakarta Validation 的支持
Spring 提供对 Jakarta Bean Validation API 的全面支持，包括将 Bean Validation provider 引导为 Spring Bean。这样，您就可以在应用程序中需要验证的地方注入 jakarta.validation.ValidatorFactory 或 jakarta.validation.Validator 。
`LocalValidatorFactoryBean` 可以实现自动加载 Jakarta Bean Validation API 的实现：
```
@Configuration
public class AppConfig {

	@Bean
	public LocalValidatorFactoryBean validator() {
		return new LocalValidatorFactoryBean();
	}
}
```
`LocalValidatorFactoryBean` 同时实现了 `jakarta.validation.Validator` 、`org.springframework.validation.Validator` 以及 `jakarta.validation.ValidatorFactory`，所以如果要使用 `Validator`，直接注入该 bean 即可：
```
@Service
public class MyService {
	@Autowired
	private jakarta.validation.Validator validator;
	@Autowired
	private org.springframework.validation.Validator validator;
}
```

## Spring MVC中的数据校验
默认情况下，如果类路径上存在 Bean Validation（例如 Hibernate Validator），则 `LocalValidatorFactoryBean` 会注册为全局 `Validator` ，以便在控制器方法参数上与 `@Valid` 和 `@Validated` 配合使用。
也可以自定义注入 `Validator` ：
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

	@Override
	public Validator getValidator() {
		// ...
	}
}
```

如果只想要在一个 controller 中使用某个特定的 `Validator` ，而不是全局注册，则可以使用如下：
```
@Controller
public class MyController {

	@InitBinder
	protected void initBinder(WebDataBinder binder) {
		binder.addValidators(new FooValidator());
	}
}
```

`@InitBinder` 是Spring MVC中的一个注解，用于在控制器中自定义数据绑定初始化的方法。它的作用主要是在处理请求之前，对请求中的数据进行初始化绑定，可以用于以下几个方面：
1. 数据预处理：通过@InitBinder注解的方法可以对表单提交的数据进行预处理，比如将日期字符串转换为Date对象、将字符串中的空格去除等。这可以确保请求数据在进入控制器方法之前已经被正确处理和转换，避免了在业务方法中重复处理数据的逻辑。
2. 数据校验：@InitBinder可以用于注册校验器（Validator），对请求中的数据进行校验。通过在@InitBinder方法中注册校验器，可以在数据绑定时对请求参数进行校验，确保数据的有效性。
3. 自定义数据绑定：有时候，Spring MVC默认的数据绑定规则可能无法满足业务需求，需要自定义数据绑定规则。通过在@InitBinder方法中注册自定义的属性编辑器（PropertyEditor），可以实现对特定类型数据的自定义绑定，比如将字符串转换为自定义对象。
4. 全局数据绑定配置：@InitBinder方法可以在控制器内部或者全局配置中使用。如果在控制器内部使用，它只对该控制器的处理方法有效；如果在全局配置中使用，则对所有的控制器都有效。






#  Spring 数据校验与绑定
{docsify-updated}

> https://docs.spring.io/spring-framework/reference/core/validation/beans-beans.html#beans-constructor-binding

## Validator 接口篇
Spring 提供了一个 `Validator` 接口，可用于验证对象。 `Validator` 接口可以向 `Errors` 对象报告验证失败。
```
public interface Validator {
	boolean supports(Class<?> clazz);
	void validate(Object target, Errors errors);
}
```
`Validator` 接口只有两个方法：
+ `boolean supports(Class<?> clazz)` : 该验证器能否验证所提供类的实例
+ `void validate(Object target, Errors errors)` : 验证一个对象，如果验证出错将错误信息放到 `Errors` 对象中

借助Spring 的 `ValidationUtils` ，实现一个 `Validator` 很简单。假设我们有一个要验证的对象类：
```
public class Customer {
	private String name;
	private int age;
	private Address address;
}
```

实现一个自定义验证器：
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

## 数据绑定
数据绑定可用于将用户输入绑定到目标对象，其中用户输入是一个以**属性路径(如account.name)**为 key 的 map。 `DataBinder` 是支持该功能的主类，它提供了两种绑定用户输入的方法：
+ 构造函数绑定: 将用户输入绑定到公共构造函数，将用户输入的参数绑定到构造函数的对应参数中。（只有高版本才支持）
+ 属性绑定：将用户输入与 `Setter` 绑定，将用户输入的键与目标对象结构的对象属性相匹配。


### 构造函数绑定
1. 创建一个以 null 为目标对象的 `DataBinder` 。
2. 将 targetType 设置为目标类。
3. 调用 `construct` 方法
```
DataBinder dataBinder = new WebDataBinder(null);
dataBinder.setTargetType(ResolvableType.forClass(A.class));
dataBinder.construct(new DataBinder.ValueResolver() {
    @Override
    public Object resolveValue(String name, Class<?> type) {
        if (name.equals("name")) return "hello";
        if (name.equals("age")) return  12;
        return null;
    }

    @Override
    public Set<String> getNames() {
        return Set.of("name","age");
    }
});
A a = (A)dataBinder.getTarget();
```

### 使用 BeanWrapper 进行属性绑定
`BeanWrapper` 接口及其相应的实现（ `BeanWrapperImpl` ）是 Bean 包中一个相当重要的类。 `BeanWrapper` 提供了设置和获取属性值（单独或批量）、获取属性描述符以及查询属性以确定其是否可读或可写的功能。此外， `BeanWrapper` 还支持嵌套属性，可在无限深度的子属性上设置属性。 `BeanWrapper` 还支持添加标准 `JavaBeans PropertyChangeListeners` 和 `VetoableChangeListeners` ，而无需在目标类中添加支持代码。最后 `BeanWrapper` 支持设置索引属性。应用程序代码通常不会直接使用 `BeanWrapper` ，但 `DataBinder` 和 `BeanFactory` 会使用它。
```
BeanWrapper company = new BeanWrapperImpl(new Company());
// setting the company name..
company.setPropertyValue("name", "Some Company Inc.");
// ... can also be done like this:
PropertyValue value = new PropertyValue("name", "Some Company Inc.");
company.setPropertyValue(value);

// ok, let's create the director and tie it to the company:
BeanWrapper jim = new BeanWrapperImpl(new Employee());
jim.setPropertyValue("name", "Jim Stravinsky");
company.setPropertyValue("managingDirector", jim.getWrappedInstance());

// retrieving the salary of the managingDirector through the company
Float salary = (Float) company.getPropertyValue("managingDirector.salary");
```
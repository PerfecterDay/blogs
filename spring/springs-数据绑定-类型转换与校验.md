#  Spring 数据绑定、类型转换与校验
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

### 属性编辑器 PropertyEditor
Spring使用PropertyEditor的概念来实现对象和字符串之间的转换。这种行为可以通过注册 `java.beans.PropertyEditor` 类型的自定义编辑器来实现。在 `BeanWrapper` 上注册自定义编辑器，或在特定 IoC 容器中注册自定义编辑器，可使其了解如何将属性转换为所需类型。  
在Spring中使用属性编辑的几个例子：
+  在Bean上设置属性是通过使用 `PropertyEditor` 的实现来完成的。当您使用 String 作为在 XML 文件中声明的某个 Bean 的属性值时，Spring（如果相应属性的设置器具有 Class 参数）会使用 `ClassEditor` 尝试将该参数解析为 `Class` 对象。
+  在Spring的MVC框架中，解析HTTP请求参数是通过使用各种 `PropertyEditor` 实现来完成的，你可以在 `CommandController` 的所有子类中手动绑`PropertyEditor`。


#### Java Bean `PropertyEditor`
Spring有许多内置的 `PropertyEditor` 实现来简化生活。它们都位于 `org.springframework.beans.propertyeditors` 包中。大多数（但不是全部，如下表所示）默认由 `BeanWrapperImpl` 注册。我们仍然可以注册你自己的变量来覆盖默认的编辑器。下表描述了Spring提供的各种 `PropertyEditor` 实现：
+ ByteArrayPropertyEditor
+ ClassEditor
+ CustomBooleanEditor
+ CustomCollectionEditor
+ CustomDateEditor
+ CustomNumberEditor
+ FileEditor
+ InputStreamEditor
+ LocaleEditor
+ PatternEditor
+ PropertiesEditor
+ StringTrimmerEditor
+ URLEditor

Spring使用 `java.beans.PropertyEditorManager` 来设置可能需要的属性编辑器的搜索路径。搜索路径还包括 `sun.bean.editors` ，其中包括 `Font` 、 `Color` 和大多数基元类型的 `PropertyEditor` 实现。还请注意，如果 `PropertyEditor` 类与它们所处理的类位于同一个包中，并且名称与该类相同，并附加了 Editor，那么标准 JavaBeans 基础架构会自动发现 `PropertyEditor` 类（无需显式注册）。例如，我们可以使用下面的类和包结构，这样就可以识别 `SomethingEditor` 类，并将其用作 `Something` 类型属性的 `PropertyEditor` ，也可以在这里使用标准的 `BeanInfo` JavaBeans 机制（在这里进行了一定程度的描述）。一般通过继承 `SimpleBeanInfo` 来是实现自己的 `BeanInfo` 。
```
com
  chank
    pop
      Something1
      Something2
      Something1Editor // the PropertyEditor for the Something class
      Something2BeanInfo // the BeanInfo for the Something class
```

如果需要注册其他自定义 `PropertyEditor` ，有几种机制可供选择。
1. 使用 `ConfigurableBeanFactory` 接口的 `registerCustomEditor()`
2. 使用 `CustomEditorConfigurer` 的特殊 `BeanFactoryPostProcessor`，直接在 `ApplicationContext` 中注册一个该类型的 bean，并将自定义 `PropertyEditor` 注册为 bean `private Map<Class<?>, Class<? extends PropertyEditor>> customEditors;` 的属性中


#### `CustomEditorConfigurer`与`PropertyEditorRegistrar`实现自定义编辑器注册
向Spring容器注册属性编辑器的另一种机制是创建并使用 `PropertyEditorRegistrar` 。当您需要在多种不同情况下使用同一组属性编辑器时，该接口尤其有用。您可以编写一个相应的注册器，并在每种情况下重复使用它。`PropertyEditorRegistrar`实例与名为 `PropertyEditorRegistry` 的接口协同工作，该接口由Spring `BeanWrapper` （和 `DataBinder`）实现。`PropertyEditorRegistrar`实例与 `CustomEditorConfigurer` （在此描述）结合使用时特别方便， `CustomEditorConfigurer` 提供了一个名为 `setPropertyEditorRegistrars(...)` 的属性。以这种方式添加到 `CustomEditorConfigurer` 的`PropertyEditorRegistrar`实例可以轻松地与 `DataBinder` 和Spring MVC控制器共享。此外，它还避免了对自定义编辑器同步的需求：`PropertyEditorRegistrar` 在每次Bean创建尝试创建新的 `PropertyEditor` 实例。
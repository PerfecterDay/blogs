# Spring 类型转换与格式化
{docsify-updated}

## 数据类型转换

### 属性编辑器 PropertyEditor
Spring使用PropertyEditor的概念来实现**对象**和**字符串**之间的转换。可以通过注册 `java.beans.PropertyEditor` 类型的自定义编辑器来实现。在 `BeanWrapper` 上注册自定义编辑器，或在特定 IoC 容器中注册自定义编辑器，可使其了解如何将属性转换为所需类型。  
在Spring中使用属性编辑的几个例子：
+  在Bean上设置属性是通过使用 `PropertyEditor` 的实现来完成的。当您使用 String 作为在 XML 文件中声明的某个 Bean 的属性值时，Spring（如果相应属性的设置器具有 Class 参数）会使用 `ClassEditor` 尝试将该参数解析为 `Class` 对象。
+  在Spring的MVC框架中，解析HTTP请求参数是通过使用各种 `PropertyEditor` 实现来完成的，你可以在 `CommandController` 的所有子类中手动绑`PropertyEditor`。

Spring有许多内置的 `PropertyEditor` 实现来简化生活。它们都位于 `org.springframework.beans.propertyeditors` 包中。大多数（但不是全部，如下表所示）默认由 `BeanWrapperImpl` 注册。可以注册你自己的变量来覆盖默认的编辑器。下表描述了Spring提供的各种 `PropertyEditor` 实现：
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

`BeanWrapperImpl`继承自 `PropertyEditorRegistrySupport` ，在 `PropertyEditorRegistrySupport` 的 `createDefaultEditors()` 方法中注册了若干的默认 `PropertyEditor` 。

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

#### 使用自定义 PropertyEditor
如果需要注册其他自定义 `PropertyEditor` ，有几种机制可供选择。
1. 如果能获取到 `BeanFactory` 的引用，可以使用 `ConfigurableBeanFactory` 接口的 `registerCustomEditor()`
2. 使用 `CustomEditorConfigurer` 的特殊 `BeanFactoryPostProcessor`，直接在 `ApplicationContext` 中注册一个该类型的 bean，并通过`setCustomEditors(Map<Class<?>, Class<? extends PropertyEditor>> customEditors)`方法将自定义一些 `PropertyEditor` 添加到 `private Map<Class<?>, Class<? extends PropertyEditor>> customEditors` 的属性中即可


##### CustomEditorConfigurer与PropertyEditorRegistrar
向Spring容器注册属性编辑器的另一种机制是创建并使用 `PropertyEditorRegistrar` 。当您需要在多种不同情况下使用同一组属性编辑器时，该接口尤其有用。可以编写一个相应的注册器，并在每种情况下重复使用它。`PropertyEditorRegistrar`实例与名为 `PropertyEditorRegistry` 的接口协同工作，该接口由Spring `BeanWrapper` （和 `DataBinder`）实现。`PropertyEditorRegistrar`实例与 `CustomEditorConfigurer` 结合使用时特别方便， `CustomEditorConfigurer` 提供了一个名为 `setPropertyEditorRegistrars(...)` 的属性。以这种方式添加到 `CustomEditorConfigurer` 的`PropertyEditorRegistrar`实例可以轻松地与 `DataBinder` 和Spring MVC控制器共享。此外，它还避免了对自定义编辑器同步的需求：`PropertyEditorRegistrar` 在每次Bean创建尝试创建新的 `PropertyEditor` 实例。

### Converter SPI
`core.convert` 软件包提供了一个通用的类型转换系统。该系统定义了用于实现类型转换逻辑的 SPI 和用于在运行时执行类型转换的 API。在 Spring 容器中，您可以使用该系统替代 `PropertyEditor` 实现，将外部化的 Bean 属性值字符串转换为所需的属性类型。您还可以在应用程序中需要进行类型转换的任何地方使用公共 API。
```
public interface Converter<S, T> {
	T convert(S source);
}
```
要创建自己的 `Converter` ，只需实现 `Converter` 接口，并将 S 设为要转换的类型，将 T 设为要转换的目标类型。如果需要将 S 的集合或数组转换为 T 的数组或集合，也可以透明地应用这种 `Converter` ，前提是委托的数组或集合 `Converter` 也已注册（默认情况下， `DefaultConversionService` 已注册）。

### ConverterFactory
当需要集中整个类层次结构的转换逻辑时（例如，从字符串转换为枚举对象），可以实现 `ConverterFactory` ，如下例所示：
```
public interface ConverterFactory<S, R> {
	<T extends R> Converter<S, T> getConverter(Class<T> targetType);
}
```
S 为要转换的类型， R 为定义可转换目标类范围的基础类型。然后实现 `getConverter(Class<T>)` ，其中 T 是 R 的子类。
```
final class StringToEnumConverterFactory implements ConverterFactory<String, Enum> {

	public <T extends Enum> Converter<String, T> getConverter(Class<T> targetType) {
		return new StringToEnumConverter(targetType);
	}

	private final class StringToEnumConverter<T extends Enum> implements Converter<String, T> {

		private Class<T> enumType;

		public StringToEnumConverter(Class<T> enumType) {
			this.enumType = enumType;
		}

		public T convert(String source) {
			return (T) Enum.valueOf(this.enumType, source.trim());
		}
	}
}
```

### GenericConverter
如果需要复杂的 `Converter` 实现，可以考虑使用 `GenericConverter` 接口。 `GenericConverter` 具有比 `Converter` 更灵活但强类型更少的签名，**支持在多个源和目标类型之间进行转换**。此外， `GenericConverter` 还提供源和目标字段上下文，供你在实现转换逻辑时使用。这种上下文允许通过字段注解或在字段特征上声明的通用信息来驱动类型转换。下面的列表显示了 `GenericConverter` 的接口定义：
```
public interface GenericConverter {
	public Set<ConvertiblePair> getConvertibleTypes();
	Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);
}
```
要实现 `GenericConverter` ，可让 `getConvertibleTypes()` 返回支持的**源→目标类型对**。然后实现 `convert(Object,TypeDescriptor,TypeDescriptor)` ，以包含转换逻辑。源 `TypeDescriptor` 提供对持有被转换值的源字段的访问。目标 `TypeDescriptor` 用于访问要设置转换值的目标字段。

参照实现： `ArrayToCollectionConverter` 

### ConditionalGenericConverter
有时，我们希望 `Converter` 仅在特定条件为真时运行。例如，只想在目标字段上存在特定注解时运行 `Converter` ，或者只想在目标类上定义了特定方法（如静态 valueOf 方法）时运行 `Converter` `。ConditionalGenericConverter` 是 `GenericConverter` 和 `ConditionalConverter` 接口的结合，可让您定义此类自定义匹配条件：
```
public interface ConditionalConverter {
	boolean matches(TypeDescriptor sourceType, TypeDescriptor targetType);
}

public interface ConditionalGenericConverter extends GenericConverter, ConditionalConverter {
}
```

参照实现： `IdToEntityConverter` 

### ConversionService 
`ConversionService` 定义了在运行时执行类型转换逻辑的统一 API。 `Converter` 通常在这些门面接口后面运行：
```
public interface ConversionService {
	boolean canConvert(Class<?> sourceType, Class<?> targetType);
	<T> T convert(Object source, Class<T> targetType);
	boolean canConvert(TypeDescriptor sourceType, TypeDescriptor targetType);
	Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);
}
```
大多数 `ConversionService` 实现还实现了 `ConverterRegistry` ，它为注册 `Converter` 提供了SPI。在内部， `ConversionService` 实现委托其注册的 `Converter` 执行类型转换逻辑。

`core.convert.support` 包中提供了强大的 `ConversionService` 实现。 `GenericConversionService` 是适用于大多数环境的通用实现。 `ConversionServiceFactory` 提供了一个方便的工厂，用于创建常用的 `ConversionService` 配置。

#### 配置 ConversionService
`ConversionService` 是一个无状态对象，设计用于在应用程序启动时实例化，然后在多个线程之间共享。在 Spring 应用程序中，通常会为每个 Spring 容器（或 ApplicationContext）配置一个 `ConversionService` 实例。只要框架需要执行类型转换，Spring 就会获取并使用该 `ConversionService` 。还可以将此 `ConversionService` 注入到您的任何 Bean 中，并直接调用它。

要向 Spring 注册默认 ConversionService，请添加以下 Bean 定义，id 为 conversionService：
```
<bean id="conversionService"
	class="org.springframework.context.support.ConversionServiceFactoryBean"/>
```
默认 `ConversionService` 可以在字符串、数字、枚举、集合、MAp和其他常见类型之间进行转换。要使用自定义 `Converter` 补充或覆盖默认 `Converter` ，请设置 `converters` 属性。属性值可以实现任何 `Converter` 、 `ConverterFactory` 或 `GenericConverter` 接口。
```
<bean id="conversionService"
		class="org.springframework.context.support.ConversionServiceFactoryBean">
	<property name="converters">
		<set>
			<bean class="example.MyCustomConverter"/>
		</set>
	</property>
</bean>
```

#### 以编程方式使用 ConversionService
要以编程方式使用 `ConversionService` 实例，可以像使用其他 Bean 一样注入一个 `ConversionService` 。下面的示例展示了如何做到这一点：
```
@Service
public class MyService {
	private final ConversionService conversionService;

	public MyService(ConversionService conversionService) {
		this.conversionService = conversionService;
	}

	public void doIt() {
		this.conversionService.convert(...)
	}
}
```

```
DefaultConversionService cs = new DefaultConversionService();
List<Integer> input = ...
cs.convert(input,
	TypeDescriptor.forObject(input), // List<Integer> type descriptor
	TypeDescriptor.collection(List.class, TypeDescriptor.valueOf(String.class)));
```

值类型的 `Converter` 可重复用于数组和集合，因此无需创建特定的 `Converter` 来将 S 的集合转换为 T 的集合。

## 字段格式化 Formatter
> https://docs.spring.io/spring-framework/reference/core/validation/format.html  
总的来说， `Formatter` 主要用于用户界面的数据格式化和解析，通常是用于针对单个字段或特定类型的格式化和解析，比如日期、货币等。而 `ConversionService` 则用于更广泛的**类型转换**，涵盖了整个应用程序的类型转换需求。
```
package org.springframework.format;
public interface Printer<T> {
	String print(T fieldValue, Locale locale); ////Java对象转换为显示数据
}

public interface Parser<T> {
	T parse(String clientValue, Locale locale) throws ParseException; //显示数据转换为Java对象
}

public interface Formatter<T> extends Printer<T>, Parser<T> {
}
```
要实现自定义的格式化 ，实现 `Formatter` 接口即可。将 T 设为要格式化的对象类型，例如 `java.util.Date` 。执行 `print()` 操作来打印 T 的实例，以便在客户端本地显示。执行 `parse()` 操作，从客户端本地语言返回的格式化表示中解析 T 的实例。如果解析尝试失败，格式器应抛出 `ParseException` 或 `IllegalArgumentException` 异常。请注意确保您的 Formatter 实现是线程安全的。

### 注解驱动的格式化
字段格式可按字段类型或注解配置。要将注解绑定到格式器，需要实现 `AnnotationFormatterFactory` 。下表显示了 AnnotationFormatterFactory 接口的定义：
```
package org.springframework.format;

public interface AnnotationFormatterFactory<A extends Annotation> {
	Set<Class<?>> getFieldTypes();
	Printer<?> getPrinter(A annotation, Class<?> fieldType);
	Parser<?> getParser(A annotation, Class<?> fieldType);
}
```
实现一个 `AnnotationFormatterFactory` 的步骤：
1. 将 A 设为要关联格式化逻辑的字段**注解**类型参数，例如 `org.springframework.format.annotation.DateTimeFormat` 。
2. 让 `getFieldTypes()` 返回可使用该注解的字段类型。
3. 让 `getPrinter()` 返回 `Printer` 将注解字段Java对象输出为格式化输出。
4. 让 `getParser()` 返回一个 `Parser` ，用于解析注解字段值到Java对象。

下面的示例 `AnnotationFormatterFactory` 实现将 `@NumberFormat` 注解与 `Formatter` 绑定，以便指定数字样式或模式：
```
public final class NumberFormatAnnotationFormatterFactory
		implements AnnotationFormatterFactory<NumberFormat> {

	private static final Set<Class<?>> FIELD_TYPES = Set.of(Short.class,
			Integer.class, Long.class, Float.class, Double.class,
			BigDecimal.class, BigInteger.class);

	public Set<Class<?>> getFieldTypes() {
		return FIELD_TYPES;
	}

	public Printer<Number> getPrinter(NumberFormat annotation, Class<?> fieldType) {
		return configureFormatterFrom(annotation, fieldType);
	}

	public Parser<Number> getParser(NumberFormat annotation, Class<?> fieldType) {
		return configureFormatterFrom(annotation, fieldType);
	}

	private Formatter<Number> configureFormatterFrom(NumberFormat annotation, Class<?> fieldType) {
		if (!annotation.pattern().isEmpty()) {
			return new NumberStyleFormatter(annotation.pattern());
		}
		// else
		return switch(annotation.style()) {
			case Style.PERCENT -> new PercentStyleFormatter();
			case Style.CURRENCY -> new CurrencyStyleFormatter();
			default -> new NumberStyleFormatter();
		};
	}
}
```
使用注解：
```
public class MyModel {
	@NumberFormat(style=Style.CURRENCY)
	private BigDecimal decimal;

	@DateTimeFormat(iso=ISO.DATE)
	private Date date;
}
```

### FormatterRegistry SPI
`FormatterRegistry` 是用于注册格式器和转换器的 SPI。 `FormattingConversionService` 是 `FormatterRegistry` 的一种实现，适用于大多数环境。以通过编程或声明的方式将此变体配置为 Spring Bean，例如使用 `FormattingConversionServiceFactoryBean` 。由于该实现也实现了 `ConversionService` ，因此可以直接将其配置为与 Spring 的 `DataBinder` 和 Spring Expression Language (SpEL) 一起使用。
```
package org.springframework.format;

public interface FormatterRegistry extends ConverterRegistry {
	void addPrinter(Printer<?> printer);
	void addParser(Parser<?> parser);
	void addFormatter(Formatter<?> formatter);
	void addFormatterForFieldType(Class<?> fieldType, Formatter<?> formatter);
	void addFormatterForFieldType(Class<?> fieldType, Printer<?> printer, Parser<?> parser);
	void addFormatterForFieldAnnotation(AnnotationFormatterFactory<? extends Annotation> annotationFormatterFactory);
}
```
可以按字段类型或注解注册格式器。 `FormatterRegistry` SPI 可让您集中配置格式化规则，而无需在各个控制器中重复配置。例如，如果希望强制所有日期字段以某种方式格式化，或强制具有特定注解的字段以某种方式格式化。有了共享的 `FormatterRegistry` ，您只需定义一次这些规则，它们就会在需要格式化时被应用。

### FormatterRegistrar SPI
```
package org.springframework.format;

public interface FormatterRegistrar {
	void registerFormatters(FormatterRegistry registry);
}
```
`FormatterRegistrar` 适用于为特定格式类别（如日期格式化）注册多个相关的 converters 和 formatters 。当声明式注册不够充分时，例如，当 formatter 需要在不同于其自身 <T> 的特定字段类型下进行索引时，或者当注册 `Printer/Parser` 。

### 全局配置日期和时间格式化
默认情况下，未注解 `@DateTimeFormat` 的日期和时间字段会使用 `DateFormat.SHORT` 样式转换为字符串。可以通过定义自己的全局格式来更改这个默认行为。
```
@Configuration
public class AppConfig {

	@Bean
	public FormattingConversionService conversionService() {

		// Use the DefaultFormattingConversionService but do not register defaults
		DefaultFormattingConversionService conversionService =
			new DefaultFormattingConversionService(false);

		// Ensure @NumberFormat is still supported
		conversionService.addFormatterForFieldAnnotation(
			new NumberFormatAnnotationFormatterFactory());

		// Register JSR-310 date conversion with a specific global format
		DateTimeFormatterRegistrar dateTimeRegistrar = new DateTimeFormatterRegistrar();
		dateTimeRegistrar.setDateFormatter(DateTimeFormatter.ofPattern("yyyyMMdd"));
		dateTimeRegistrar.registerFormatters(conversionService);

		// Register date conversion with a specific global format
		DateFormatterRegistrar dateRegistrar = new DateFormatterRegistrar();
		dateRegistrar.setFormatter(new DateFormatter("yyyyMMdd"));
		dateRegistrar.registerFormatters(conversionService);

		return conversionService;
	}
}
```
如果是 Spring Web 环境：
```
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

	@Override
	public void addFormatters(FormatterRegistry registry) {
		DateTimeFormatterRegistrar registrar = new DateTimeFormatterRegistrar();
		registrar.setUseIsoFormat(true);
		registrar.registerFormatters(registry);
     	}
}
```



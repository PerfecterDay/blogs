#  Spring 数据绑定、类型转换与校验
{docsify-updated}

- [Spring 数据绑定、类型转换与校验](#spring-数据绑定类型转换与校验)
	- [属性编辑器](#属性编辑器)
		- [Java Bean `PropertyEditor`](#java-bean-propertyeditor)
		- [`CustomEditorConfigurer`与`PropertyEditorRegistrar`实现自定义编辑器注册](#customeditorconfigurer与propertyeditorregistrar实现自定义编辑器注册)
	- [验证](#验证)


### 属性编辑器
Spring使用PropertyEditor的概念来实现对象和字符串之间的转换。 在Spring中使用属性编辑的几个例子：
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

### 验证
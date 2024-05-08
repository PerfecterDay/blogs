#  Spring IOC 概览
 {docsify-updated}
> https://docs.spring.io/spring-framework/reference/core/beans.html

`org.springframework.context.ApplicationContext` 接口代表 Spring IoC 容器，负责实例化、配置和装配 Bean。容器通过读取**配置元数据**来获取有关实例化、配置和装配哪些对象的指令。配置元数据用 XML、Java 注解或 Java 代码表示。通过它，您可以表达组成应用程序的对象以及这些对象之间丰富的相互依赖关系。

+ 基于注解的配置(`AnnotationConfigApplicationContext/AnnotationConfigWebApplicationContext`)：使用基于注释的配置元数据定义 Bean。
+ 基于 Java 的配置：使用 Java 而不是 XML 文件定义应用程序类外部的 Bean。要使用这些功能，通常使用 `@Configuration`、`@Bean`、`@Import` 和 `@DependsOn` 注解。
+ 基于XML文件的配置(`ClassPathXmlApplicationContext/FileSystemXmlApplicationContext`)

Spring 提供了多个 `ApplicationContext` 接口的实现。在独立应用程序中，创建 `ClassPathXmlApplicationContext` 或 `FileSystemXmlApplicationContext` 的实例很常见。虽然 XML 一直是定义配置元数据的传统格式，但您可以通过提供少量 XML 配置来声明启用对这些附加元数据格式的支持，从而指示容器使用 **Java 注解**或**Java 代码**作为元数据格式。

## 使用步骤
1. 配置好容器
2. 实例化一个 Container
```
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```
3. 使用容器
```
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

## Bean 概览
在容器内部， Bean 定义表示为 `BeanDefinition` 对象，其中包含（除其他信息外）以下元数据：
+ 一个带包全路径的类名：定义的Bean的实际实现类。
+ Bean的行为配置元素，它说明了Bean在容器中的行为方式（范围、生命周期回调等等）。
+ 对其他Bean的引用，Bean需要依赖这些其他Bean实现它的功能。这些引用也被称为依赖关系或合作者。
+ Bean属性的配置--例如，在管理连接池的Bean中，池的大小限制或使用的连接数配置。

这些元数据转化为组成每个 Bean 定义的一系列属性。下表描述了这些属性：
+ `Class`
+ `Name`
+ `Scope`
+ `Constructor arguments`
+ `Properties`
+ `Autowiring mode`
+ `Lazy initialization mode`
+ `Initialization method`
+ `Destruction method`

`ApplicationContext` 还允许注册在容器外（由用户）创建的现有对象。这是通过 `getBeanFactory()` 方法访问 `ApplicationContext` 的 `BeanFactory` 来实现的，该方法返回 `DefaultListableBeanFactory` 实现。 `DefaultListableBeanFactory` 通过 `registerSingleton(.)` 和 `registerBeanDefinition(.)` 方法支持这种注册。然而，典型的应用程序通常通过常规Bean定义方式声明Bean。

如果使用基于 XML 的配置元数据，则应在 <bean/> 元素的 class 属性中指定要实例化的对象类型（或类别）。这个 class 属性（在内部是 BeanDefinition 实例的 Class 属性）通常是强制性的。(例外情况是使用实例工厂方法和 Bean 定义继承进行实例化）。您可以通过以下两种方式之一使用 Class 属性：
+ 通常，在容器本身通过调用构造器直接创建 bean 的情况下，指定要构造的 bean 类，这有点类似于使用 new 操作符的 Java 代码。
+ 在容器调用一个类上的静态工厂方法来创建 Bean 这种不太常见的情况下，指定包含调用静态工厂方法来创建对象的实际类。调用静态工厂方法返回的对象类型可能是同一个类，也可能完全是另一个类。

声明内部嵌套类：
例如，如果您在 com.example 包中有一个名为 SomeThing 的类，而这个 SomeThing 类又有一个名为 OtherThing 的静态嵌套类，那么它们之间可以用美元符号（\$）或点（.）隔开。因此，Bean 定义中 class 属性的值将是 `com.example.SomeThing$OtherThing` 或 `com.example.SomeThing.OtherThing`。

### Bean 实例化

#### 通过构造方法实例化
当您使用构造函数方法创建 Bean 时，所有普通类都可以被 Spring 使用并与 Spring 兼容。也就是说，开发的类不需要实现任何特定的接口，也不需要以特定的方式编码。只需指定 bean 类即可。不过，根据您为特定 Bean 使用的 IoC 类型，您可能需要一个默认（空）构造函数。
```
<bean id="exampleBean" class="examples.ExampleBean"/>
<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

#### 使用静态工厂方法进行实例化
在定义使用静态工厂方法创建的 Bean 时，请使用 class 属性指定包含静态工厂方法的类，并使用名为 `factory-method` 的属性指定工厂方法本身的名称。
```
<bean id="clientService"
	class="examples.ClientService"
	factory-method="createInstance"/>

public class ClientService {
	private static ClientService clientService = new ClientService();
	private ClientService() {}

	public static ClientService createInstance() {
		return clientService;
	}
}
```
#### 使用实例工厂方法进行实例化
与通过静态工厂方法进行实例化类似，使用实例工厂方法进行实例化时，会调用容器中现有 Bean 的一个非静态方法来创建一个新 Bean。要使用这种机制，请将 class 属性留空，并在 `factory-bean` 属性中指定当前（或父或祖）容器中包含实例方法的 bean 的名称，该实例方法将被调用来创建对象。使用 `factory-method` 属性设置工厂方法本身的名称。下面的示例展示了如何配置这样一个 Bean：
```
<!-- the factory bean, which contains a method called createClientServiceInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
	<!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
	factory-bean="serviceLocator"
	factory-method="createClientServiceInstance"/>

public class DefaultServiceLocator {

	private static ClientService clientService = new ClientServiceImpl();

	public ClientService createClientServiceInstance() {
		return clientService;
	}
}
```

### 确定 bean 的运行时类型
要确定一个特定Bean的运行时类型是不容易的。在Bean元数据定义中指定的类只是一个初始的类引用，可能与已声明的工厂方法相结合，或者是一个FactoryBean类，这可能导致Bean的运行时类型不同，或者在实例级工厂方法的情况下根本没有被设置（而是通过指定的工厂-bean名称来解决）。此外，AOP代理可能会用基于接口的代理来包装Bean实例，对目标Bean的实际类型（只是其实现的接口）的暴露有限。

要了解某个特定Bean的实际运行时类型，建议使用 `BeanFactory.getType(String beanName)` 方法或者指定bean的类型。这将考虑到上述所有情况，并返回 `BeanFactory.getBean` 调用对同一Bean名称将返回的对象类型。

### Bean 生命周期回调
为了与容器对Bean生命周期的管理进行交互，你可以实现Spring `InitializingBean` 和 `DisposableBean` 接口。容器为前者调用 `afterPropertiesSet()` ，为后者调用 `destroy()` ，让Bean在初始化和销毁你的Bean时执行某些动作。如果你的bean 需要在 Spring 为你设置好某些属性或者在销毁前做一些事情，可以实现这些接口。

JSR-250的 `@PostConstruct` 和 `@PreDestroy` 注解通常被认为是在现代Spring应用程序中接收生命周期回调的最佳实践。使用这些注解意味着你的Bean不会被耦合到Spring特定的接口。

### 在非网络应用中优雅地关闭Spring IoC容器
如果你在非网络应用环境中使用Spring的IoC容器（例如，在富客户端桌面环境中），请向JVM注册一个关机钩。这样做可以确保优雅地关闭，并在你的单体Bean上调用相关的destroy方法，从而释放所有资源。你仍然必须正确配置和实现这些销毁回调。

要注册一个关机钩子，请调用`registerShutdownHook()`方法，该方法在`ConfigurableApplicationContext`接口上声明，如下面的例子所示:
```
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

	public static void main(final String[] args) throws Exception {
		ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

		// add a shutdown hook for the above context...
		ctx.registerShutdownHook();

		// app runs here...

		// main method exits, hook is called prior to the app shutting down...
	}
}
```

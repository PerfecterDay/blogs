## Spring IOC BeanDefinition
 {docsify-updated}
> https://docs.spring.io/spring-framework/reference/core/beans.html

- [Spring IOC BeanDefinition](#spring-ioc-beandefinition)
	- [BeanDefinition](#beandefinition)
	- [Bean 生命周期回调](#bean-生命周期回调)
	- [在非网络应用中优雅地关闭Spring IoC容器](#在非网络应用中优雅地关闭spring-ioc容器)


### BeanDefinition
+ 一个带包全路径的类名：定义的Bean的实际实现类。
+ Bean的行为配置元素，它说明了Bean在容器中的行为方式（范围、生命周期回调等等）。
+ 对其他Bean的引用，Bean需要依赖这些其他Bean实现它的功能。这些引用也被称为合作者或依赖关系。
+ 在Bean属性的配置--例如，在管理连接池的Bean中，池子的大小限制或使用的连接数。


`ApplicationContext` 还允许注册在容器外（由用户）创建的现有对象。这是通过 `getBeanFactory()` 方法访问 `ApplicationContext` 的 `BeanFactory` 来实现的，该方法返回 `DefaultListableBeanFactory` 实现。 `DefaultListableBeanFactory` 通过 `registerSingleton(.)` 和 `registerBeanDefinition(.)` 方法支持这种注册。然而，典型的应用程序通常通过常规Bean定义方式声明Bean。


声明内部嵌套类： `com.example.SomeThing.OtherThing` 或者 `com.example.SomeThing$OtherThing`


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

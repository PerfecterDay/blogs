# Bean scopes
{docsify-updated}

创建一个 `BeanDefinition` 时，你实际上是在创建一个如何创建 Bean 的配方。将 `BeanDefinition` 视为配方这一概念至关重要，因为这意味着如同Java类一样，我们能创建一个类的多个实例对象，同样， Spring 也可以通过 `BeanDefinition` 配方生成多个对象实例 Bean。

您不仅能够控制从特定Bean定义创建的对象中需要注入的各种依赖项和配置值，还能控制从特定Bean定义创建的对象的作用域。这种方法既强大又灵活，因为您可以通过配置选择创建对象的作用域，而无需在Java类级别硬编码对象的作用域。Bean可以被定义为部署在多种作用域之一中。Spring框架支持六种作用域，其中四种仅在使用Web感类型的 `ApplicationContext` 时可用。

+ `singleton`: 标记为拥有 `singleton scope` 的 `BeanDefinition`，在Spring的IoC容器中只存在一个实例，所有对该对象的引用将共享这个实例。该实例从容器启动，并因为第一次被请求而初始化之后，将一直存活到容器退出，也就是说，它与IoC容器“几乎”拥有相同的生命周期。
+ `prototype` ：容器在接到该类型对象的请求的时候，会每次都重新生成一个新的对象实例给请求方。虽然这种类型的对象的实例化以及属性设置等工作都是由容器负责的，但是只要准备完毕，并且对象实例返回给请求方之后，容器就不再拥有当前返回对象的引用，请求方需要自己负责当前返回对象的后继生命周期的管理工作，包括该对象的销毁。也就是说，容器每次返回给请求方一个新的对象实例之后，就任由这个对象实例“自生自灭”了。
+ `request`: Spring容器会为每个HTTP请求创建一个全新的bean对象供当前请求使用，当请求结束后，该对象实例的生命周期即告结束。
+ `session` ：Spring容器会为每个独立的 `session` 创建属于它们自己的全新的bean对象实例。与 `request` 相比，除了可能更长的存活时间，其他方没什么差别。
+ `application`: 将单个 Bean 定义的作用域扩展到 `ServletContext` 的生命周期。仅在Web 类型的 `Spring ApplicationContext` 的上下文中有效。
+ `websocket`: 将单个 Bean 定义的作用域扩展到 `WebSocket` 的生命周期。仅在Web 类型的 `Spring ApplicationContext` 的上下文中有效。


## Singleton
<center><img src="pics/singleton.png" alt=""></center>

仅管理单例 bean 的一个共享实例，所有请求 ID 与该 bean 定义匹配的 bean 请求，都会由 Spring 容器返回该特定 bean 实例。换言之，当你定义一个 `singleton scope` 的Bean时，Spring IoC容器会为该Bean定义创建且仅创建一个对象实例。这个单一实例会被存储在单例Bean的缓存中，此后所有对该命名Bean的请求和引用都会返回缓存中的对象。

若在单个Spring容器中为特定类定义一个Bean，则该容器仅会创建该Bean定义所对应类的唯一实例。单例范围是Spring的默认作用域。


## Prototype
<center><img src="pics/prototype.png" alt=""></center>

当 `prototype` 的 bean被注入到另一个bean中，或通过容器的 `getBean()` 方法调用请求时，都会触发实例创建。通常应遵循以下原则：**所有有状态bean应采用 `prototype scope` ，无状态bean则应采用 `singleton scope` 。**

与其他作用域不同，Spring 不会管理原型 Bean 的完整生命周期。容器会实例化、配置并组装原型对象，然后将其传递给应用程序后，就不会对该原型实例进行后续记录。因此，尽管初始化生命周期回调方法会调用所有对象（无论作用域如何），但在 `prototype` 情况下，**配置的销毁生命周期回调不会被调用。客户端代码必须自行清理 `prototype scope` 对象，并释放原型 bean 占用的高成本资源**。若需让 Spring 容器释放 `prototype scope`  bean 占用的资源，可尝试使用自定义 `bean post-processor` ，该处理器会持有待清理 bean 的引用。

从某些方面来看，Spring容器对原型作用域Bean的作用相当于Java new运算符的替代方案。此后所有生命周期管理都必须由客户端自行处理。


### signleton 依赖 prototype 
当使用依赖原型 bean 的单例作用域 bean 时，请注意依赖关系在实例化时解决。因此，若将原型作用域 bean 依赖注入到单例作用域 bean 中，系统会先实例化新的原型 bean，再将其注入单例 bean。该原型实例将成为唯一提供给单例作用域 bean 的实例。

然而，假设你希望单例作用域的bean在运行时反复获取原型作用域bean的新实例。你无法将原型作用域bean依赖注入到单例bean中，因为这种注入仅发生一次——当Spring容器实例化单例bean并解析注入其依赖项时。若需在运行时多次获取原型bean的新实例，请参阅方法注入。


## Request, Session, Application, and WebSocket Scopes
`request` , `session` , `application` 和 `websocket` scopes 仅在使用支持 Web 的Spring `ApplicationContext` 实现（如 `XmlWebApplicationContext` ）时可用。若在常规Spring IoC容器（如 `ClassPathXmlApplicationContext`）中使用这些作用域，将抛出 `IllegalStateException` 异常，提示存在未知bean作用域。

### 初始化 Web 配置
为支持在 `request` , `session` , `application` 和 `websocket` 级别（即Web作用域Bean）定义Bean的作用域，您需要在定义Bean之前进行一些简单的初始配置。（单例和原型则无需进行。）

+ 在Spring Web MVC中访问作用域bean时，实际上是在Spring `DispatcherServlet` 处理的请求内进行操作，因此无需特殊配置。 `DispatcherServlet` 已自动暴露所有相关状态。
+ 若使用Servlet Web容器且请求在Spring的DispatcherServlet之外处理（例如使用JSF时），则需注册org.springframework.web.context.request.RequestContextListener ServletRequestListener。可通过WebApplicationInitializer接口以编程方式实现，或在Web应用程序的web.xml文件中添加以下声明：
  ```
  <web-app>
      ...
      <listener>
          <listener-class>
              org.springframework.web.context.request.RequestContextListener
          </listener-class>
      </listener>
      ...
  </web-app>
  ```
+ 或者，如果监听器配置存在问题，请考虑使用 Spring 的 `RequestContextFilter` 。该过滤器的映射取决于周围的 Web 应用程序配置，因此您需要根据实际情况进行调整。以下代码片段展示了 Web 应用程序中的过滤器部分：
  ```
  <web-app>
  	...
  	<filter>
  		<filter-name>requestContextFilter</filter-name>
  		<filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
  	</filter>
  	<filter-mapping>
  		<filter-name>requestContextFilter</filter-name>
  		<url-pattern>/*</url-pattern>
  	</filter-mapping>
  	...
  </web-app>
  ```

`DispatcherServlet` 、 `RequestContextListener` 和 `RequestContextFilter` 的功能完全相同，即把 HTTP 请求对象绑定到处理该请求的线程。这使得请求作用域和会话作用域的 Bean 在调用链的后续环节中可用。


### Request scope
```
<bean id="loginAction" class="com.something.LoginAction" scope="request"/>

@RequestScope
@Component
public class LoginAction {
	// ...
}
```
Spring容器会为每个HTTP请求使用loginAction Bean定义创建一个新的LoginAction Bean实例。也就是说，loginAction Bean的作用域限定在HTTP请求级别。您可以随意修改创建实例的内部状态，因为其他从同一loginAction bean定义创建的实例不会看到这些状态变化。这些变化仅针对单个请求。当请求处理完成时，作用域限定于该请求的bean会被丢弃。

### Session Scope
```
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>

@SessionScope
@Component
public class UserPreferences {
	// ...
}
```
Spring容器通过 userPreferences Bean定义创建UserPreferences Bean的新实例，其生命周期仅限于单次HTTP会话。换言之，userPreferences Bean的有效作用域限定在HTTP会话级别。与请求作用域的 Bean 类似，您可以随意修改所创建实例的内部状态，但需注意：其他使用相同 userPreferences Bean 定义创建的实例不会看到这些状态变更，因为这些变更仅属于特定的 HTTP 会话。当 HTTP 会话最终被丢弃时，属于该特定 HTTP 会话作用域的 Bean 也将被丢弃。

### Application Scope
```
<bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>

@ApplicationScope
@Component
public class AppPreferences {
	// ...
}
```

Spring容器通过在整个Web应用程序中仅使用一次appPreferences Bean定义来创建AppPreferences Bean的新实例。也就是说，appPreferences Bean的作用域限定在 `ServletContext` 级别，并作为常规的 `ServletContext` 属性存储。这与Spring单例bean有相似之处，但存在两点重要差异：
+ 其一，它是按 `ServletContext` 实例化的单例（而非按Spring `ApplicationContext` 实例化——单个Web应用中可能存在多个 `ApplicationContext`）；
+ 其二，它实际暴露为 `ServletContext` 属性，因此可见。

### WebSocket Scope
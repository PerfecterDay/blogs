#  spring 事件机制
{docsify-updated}

> https://docs.spring.io/spring-framework/reference/core/beans/context-introduction.html
- [spring 事件机制](#spring-事件机制)
		- [Java 观察者模式（事件机制）](#java-观察者模式事件机制)
		- [spring 事件体系](#spring-事件体系)
		- [Spring事件体系的具体实现](#spring事件体系的具体实现)
		- [Spring自带的发布事件](#spring自带的发布事件)
		- [实现自己的业务事件发布与监听](#实现自己的业务事件发布与监听)


Spring 的 ApplicationContext 能够发布事件并且允许注册相应的事件监听器，因此，它拥有一套完善的事件发布和监听机制。在 Java 中， `java.util.EventObject` 类和 `java.util.EventListener` 接口描述了事件和监听器。在事件体系中，除了事件和监听器外，还有另外三个重要概念。

+ 事件源：事件的生产者，任何一个 `EventObject` 都必须有一个事件源。
+ 事件监听器注册表：组件或框架的事件监听器不可漂浮在空中，而必须有所依存。也就是说组件或框架必须提供一个地方保存事件监听器，这便是事件监听器注册表。当组件或框架中的事件源产生事件时，就会通知位于事件监听器注册表中的监听器。
+ 事件广播器：它是事件和事件监听器沟通的桥梁，负责把事件通知给事件监听器。

事件源、事件监听器注册表和事件广播器这3个角色有时可以由同一个对象承担，其实就是观察者模式中的 `Observable` ，监听器就类似于 `Observer` 。事件体系其实是观察者模式的一种具体实现方式。

### Java 观察者模式（事件机制）
<center><img src="pics/java-event.png" width=45%></center>

### spring 事件体系
<center><img src="pics/spring-event.png" width=55%></center>

可以根据需要扩展 `ApplicationEvent` 定义自己的事件。

### Spring事件体系的具体实现
Spring 在 `ApplicationContext` 接口的抽象实现类 `AbstratApplicationContext`中完成事件体系的搭建。`AbstratApplicationContext`中拥有一个 `ApplicationEventMulticaster` 的成员，该成员提供了容器监听器的注册表。`AbstratApplicationContext`在 `refresh()` 这个容器启动方法中通过以下3个步骤搭建了事件的基础设施：
```
..
initApplicationEventMulticaster();
...
registerListeners();
...
finishRefresh();
```
首先初始化事件广播器，用户可以在Spring配置中定义一个事件广播器，只要实现 `ApplicationEventMulticater` 即可， spring 会通过反射机制将其注册成容器的事件广播器，如果没有找到配置的外部事件广播器， Spring 自动使用 `SimpleApplicationEventMulticaster` 作为事件广播器。

然后， Spring 根据反射机制，从注册的 bean 中，找出所有实现了 `ApplicationListener` 的 bean ，并将它们注册为容器的事件监听器，实际操作就是将其添加到事件广播器所提供的事件监听器注册表中。

最后，调用容器的事件发布接口 **`publishEvent()`** 向容器中所有的监听器发布事件。所谓的发布事件本质上其实就是构造一个事件，然后循环调用 `ApplicationListener`的 `onApplicationEvent`方法：
```
@Override
public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
	ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
	Executor executor = getTaskExecutor();
	for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
		if (executor != null) {
			executor.execute(() -> invokeListener(listener, event));
		}
		else {
			invokeListener(listener, event);
		}
	}
}
```

### Spring自带的发布事件

| Event | Explanation |
| ----------- | ----------- |
| ContextRefreshedEvent | Published when the ApplicationContext is initialized or refreshed (for example, by using the refresh() method on the ConfigurableApplicationContext interface). Here, “initialized” means that all beans are loaded, post-processor beans are detected and activated, singletons are pre-instantiated, and the ApplicationContext object is ready for use. As long as the context has not been closed, a refresh can be triggered multiple times, provided that the chosen ApplicationContext actually supports such “hot” refreshes. For example, XmlWebApplicationContext supports hot refreshes, but GenericApplicationContext does not. |
| ContextStartedEvent | Published when the ApplicationContext is started by using the start() method on the ConfigurableApplicationContext interface. Here, “started” means that all Lifecycle beans receive an explicit start signal. Typically, this signal is used to restart beans after an explicit stop, but it may also be used to start components that have not been configured for autostart (for example, components that have not already started on initialization). |
| ContextStoppedEvent | Published when the ApplicationContext is stopped by using the stop() method on the ConfigurableApplicationContext interface. Here, “stopped” means that all Lifecycle beans receive an explicit stop signal. A stopped context may be restarted through a start() call. |
| ContextClosedEvent | Published when the ApplicationContext is being closed by using the close() method on the ConfigurableApplicationContext interface or via a JVM shutdown hook. Here, "closed" means that all singleton beans will be destroyed. Once the context is closed, it reaches its end of life and cannot be refreshed or restarted. |
| RequestHandledEvent | A web-specific event telling all beans that an HTTP request has been serviced. This event is published after the request is complete. This event is only applicable to web applications that use Spring’s DispatcherServlet. |
| ServletRequestHandledEvent | A subclass of RequestHandledEvent that adds Servlet-specific context information. |

### 手动实现自己的业务事件发布与监听
1. 定义业务事件，继承自 `ApplicationEvent` 
2. 发布者（需要发布自定义业务事件的业务Bean）实现 `ApplicationEventPublisherAware`,`ApplicationContextAware` 接口，利用 `ApplicationContext` 对象就可以发布自定义的事件
3. 实现 `ApplicationListener` 接口并注册到容器中，实现监听自定义事件逻辑
4. 或者使用 `@EventListener` 注解：
   ```
	@Component
	public class AnnotationDrivenEventListener {
		@EventListener
		public void handleContextStart(ContextStartedEvent cse) {
			System.out.println("Handling context started event.");
		}
	}
   ```
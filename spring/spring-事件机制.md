# spring 事件机制
{docsify-updated}

Spring 的 ApplicationContext 能够发布事件并且允许注册相应的事件监听器，因此，它拥有一套完善的事件发布和监听机制。在 Java 中， `java.util.EventObject` 类和 `java.util.EventListener` 接口描述了事件和监听器。在事件体系中，除了事件和监听器外，还有另外三个重要概念。

+ 事件源：事件的生产者，任何一个 `EventObject` 都必须有一个事件源。
+ 事件监听器注册表：组件或框架的事件监听器不可漂浮在空中，而必须有所依存。也就是说组件或框架必须提供一个地方保存事件监听器，这便是事件监听器注册表。当组件或框架中的事件源产生事件时，就会通知位于事件监听器注册表中的监听器。
+ 事件广播器：它是事件和事件监听器沟通的桥梁，负责把事件通知给事件监听器。

事件源、事件监听器注册表和事件广播器这3个角色有时可以由同一个对象承担，其实就是观察者模式中的 `Observable` ，监听器就类似于 `Observer` 。事件体系其实是观察者模式的一种具体实现方式。

### spring 事件类
![Spring 事件类](../pics/spring-event.png)

可以根据需要扩展 `ApplicationEvent` 定义自己的事件。

### spring 事件监听器接口
![Spring 事件监听器](../pics/spring-listener.png)

### spring 事件广播器
![Spring 事件广播器](../pics/spring-multicaster.png)

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

最后，调用容器的事件发布接口 **`publishEvent()`** 向容器中所有的监听器发布事件。


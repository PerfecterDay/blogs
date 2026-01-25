# Bean scopes
{docsify-updated}

创建一个 `BeanDefinition` 时，你实际上是在创建一个如何创建 Bean 的配方。将 `BeanDefinition` 视为配方这一概念至关重要，因为这意味着如同Java类一样，我们能创建一个类的多个实例对象，同样， Spring 也可以通过 `BeanDefinition` 配方生成多个对象实例 Bean。

您不仅能够控制从特定Bean定义创建的对象中需要注入的各种依赖项和配置值，还能控制从特定Bean定义创建的对象的作用域。这种方法既强大又灵活，因为您可以通过配置选择创建对象的作用域，而无需在Java类级别硬编码对象的作用域。Bean可以被定义为部署在多种作用域之一中。Spring框架支持六种作用域，其中四种仅在使用Web感类型的 `ApplicationContext` 时可用。

+ `singleton`: 标记为拥有singleton scope的对象定义，在Spring的IoC容器中只存在一个实例，所有对该对象的引用将共享这个实例。该实例从容器启动，并因为第一次被请求而初始化之后，将一直存活到容器退出，也就是说，它与IoC容器“几乎”拥有相同的“寿命”。
+ `prototype` ：容器在接到该类型对象的请求的时候，会每次都重新生成一个新的对象实例给请求方。虽然这种类型的对象的实例化以及属性设置等工作都是由容器负责的，但是只要准备完毕，并且对象实例返回给请求方之后，容器就不再拥有当前返回对象的引用，请求方需要自己负责当前返回对象的后继生命周期的管理工作，包括该对象的销毁。也就是说，容器每次返回给请求方一个新的对象实例之后，就任由这个对象实例“自生自灭”了。
+ `request`: Spring容器会为每个HTTP请求创建一个全新的bean对象供当前请求使用，当请求结束后，该对象实例的生命周期即告结束。
+ `session` ：Spring容器会为每个独立的session创建属于它们自己的全新的bean对象实例。与request相比，除了可能更长的存活时间，其他方面真是没什么差别。
+ `application`: 将单个 Bean 定义的作用域扩展到 ServletContext 的生命周期。仅在Web 类型的 Spring ApplicationContext 的上下文中有效。
+ `websocket`: 将单个 Bean 定义的作用域扩展到 WebSocket 的生命周期。仅在Web 类型的 Spring ApplicationContext 的上下文中有效。

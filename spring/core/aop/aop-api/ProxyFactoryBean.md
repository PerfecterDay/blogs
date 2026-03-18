# 使用 ProxyFactoryBean 创建 AOP 代理
{docsify-updated}

> https://docs.spring.io/spring-framework/reference/core/aop-api/pfb.html

如果在业务对象中使用了 Spring IoC 容器（即 `ApplicationContext` 或 `BeanFactory` ）（你确实应该这样做），那么你应该使用 Spring 的 AOP `FactoryBean` 实现之一。（请记住，工厂 Bean 引入了一层间接层，使其能够创建不同类型的对象。）

在 Spring 中创建 AOP 代理的基本方法是使用 `org.springframework.aop.framework.ProxyFactoryBean` 。这可以让你完全控制切点、适用的任何 `advice` 以及它们的执行顺序。不过，如果你不需要这种控制，还有一些更简单的选项值得优先考虑。

## 基础
与其他 Spring FactoryBean 实现类似， `ProxyFactoryBean` 引入了一层间接关系。如果你定义了一个名为 `foo` 的 `ProxyFactoryBean` ，引用 `foo` 的对象看到的并非 `ProxyFactoryBean` 实例本身，而是由 `ProxyFactoryBean` 中 `getObject()` 方法的实现所创建的对象。该方法会创建一个 AOP 代理，用于封装目标对象。

使用 `ProxyFactoryBean` 或其他支持 IoC 的类来创建 AOP 代理的一个最重要优势在于， `advice` 和切入点也可以由 IoC 进行管理。这是一个强大的功能，它支持某些在其他 AOP 框架中难以实现的实现方式。例如，一个 `advice` 本身可以引用应用程序对象（除了目标对象——该对象在任何 AOP 框架中都应可用），从而充分利用依赖注入所提供的所有可插拔性。


## JavaBean Properties
与 Spring 提供的绝大多数 `FactoryBean` 实现一样， `ProxyFactoryBean` 类本身也是一个 JavaBean。其属性用于：
+ 指定要代理的目标
+ 指定是否使用 `CGLIB`

某些关键属性是从 `org.springframework.aop.framework.ProxyConfig`（Spring 中所有 AOP 代理工厂的超类）继承而来的。这些关键属性包括以下内容：
+ `proxyTargetClass` : 如果要代理的是目标**类**本身（而非目标类的**接口**），则该值为 `true` 。如果将此属性值设置为 `true` ，则会创建 `CGLIB` 代理（代理方式有基于 JDK 和 CGLIB 的代理）。
+ `optimize` : 控制是否对通过 `CGLIB` 创建的代理应用激进的优化。除非您完全了解相关 AOP 代理如何处理优化，否则不应轻率地使用此设置。目前该设置仅适用于 `CGLIB` 代理。对于 JDK 动态代理，此设置无效。
+ `frozen` : 如果代理配置处于冻结状态，则不再允许对其进行修改。此功能既可作为轻微的性能优化，也适用于不希望调用方在代理创建后（通过 `Advised` 接口）对其进行操作的情况。该属性的默认值为 `false` ，因此允许进行修改（例如添加额外的 `advice` ）。
+ `exposeProxy` : 决定是否应将当前代理暴露在 `ThreadLocal` 中，以便目标能够访问它。如果目标需要获取代理，且 `exposeProxy` 属性设置为 true，则目标可以使用 `AopContext.currentProxy()` 方法获取。
+ `proxyInterfaces` ：一个包含字符串接口名称的数组。如果未提供此参数，则使用目标类的 `CGLIB` 代理.
+ `interceptorNames` ：一个字符串数组，包含要应用的 `Advisor` 、 `interceptor` 或其他 `advice` 的名称。顺序很重要，遵循先到先得的原则。也就是说，列表中的第一个拦截器将最先拦截该调用。
    这些名称是当前工厂中的 Bean 名称，包括来自祖先工厂的 Bean 名称。此处不能直接引用 Bean，因为这样做会导致 `ProxyFactoryBean` 忽略 `advice` 的 `singleton` 设置。
    可以在拦截器名称后添加星号（`*`）。这样做将导致所有名称以星号前部分开头的 advisor Bean 被应用。
+ `singleton` ：无论 `getObject()` 方法被调用多少次，工厂是否应返回单个对象。一些 `FactoryBean` 实现提供了此方法。默认值为 `true` 。若要使用带状态的 `advice` （例如，用于带状态的混入），请将 `prototype`  `advice` 与 `singleton` 值 set 为 `false` 结合使用。


## JDK and CGLIB-based proxies
本节详细说明 `ProxyFactoryBean` 如何为特定的目标对象（即待代理的对象）选择创建基于JDK的代理或基于CGLIB的代理。

如果待代理的目标对象的类（以下简称目标类）未实现任何接口，则会创建基于 `CGLIB` 的代理。这是最简单的情况，因为 JDK 代理是基于接口的，而没有接口就意味着无法进行 JDK 代理。可以通过设置 `interceptorNames` 属性来注入目标 Bean 并指定拦截器列表。请注意，即使 `ProxyFactoryBean` 的 `proxyTargetClass` 属性被设置为 `false` ，系统仍会创建基于 CGLIB 的代理。（这样做毫无意义，最好从 Bean 定义中移除该属性，因为它充其量是多余的，最坏的情况下会造成混淆。）

如果目标类实现了（一个或多个）接口，则创建的代理类型取决于 `ProxyFactoryBean` 的配置。

如果 `ProxyFactoryBean` 的 `proxyTargetClass` 属性被设置为 `true` ，则会创建一个基于 `CGLIB` 的代理。这合乎逻辑，也符合“最小意外原则”。即使 `ProxyFactoryBean` 的 `proxyInterfaces` 属性被设置为一个或多个完全限定的接口名称，只要 `proxyTargetClass` 属性被设置为 `true` ，基于 `CGLIB` 的代理机制就会生效。

如果 `ProxyFactoryBean` 的 `proxyInterfaces` 属性被设置为一个或多个完全限定的接口名称，则会创建一个基于 JDK 的代理。生成的代理将实现 `proxyInterfaces` 属性中指定的所有接口。如果目标类恰好实现了比 proxyInterfaces 属性中指定接口数量多得多的接口，这当然没问题，但返回的代理不会实现这些额外的接口。
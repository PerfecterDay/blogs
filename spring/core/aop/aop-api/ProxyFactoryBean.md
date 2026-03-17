# 使用 ProxyFactoryBean 创建 AOP 代理
{docsify-updated}

> https://docs.spring.io/spring-framework/reference/core/aop-api/pfb.html

如果在业务对象中使用了 Spring IoC 容器（即 `ApplicationContext` 或 `BeanFactory` ）（你确实应该这样做），那么你应该使用 Spring 的 AOP `FactoryBean` 实现之一。（请记住，工厂 Bean 引入了一层间接层，使其能够创建不同类型的对象。）

在 Spring 中创建 AOP 代理的基本方法是使用 `org.springframework.aop.framework.ProxyFactoryBean` 。这可以让你完全控制切点、适用的任何建议以及它们的执行顺序。不过，如果你不需要这种控制，还有一些更简单的选项值得优先考虑。


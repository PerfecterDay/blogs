# Classpath Scanning and Managed Components
{docsify-updated}

> https://docs.spring.io/spring-framework/reference/core/beans/classpath-scanning.html

本章中的多数示例均采用XML来指定生成Spring容器内每个 `BeanDefinition` 的配置元数据。前一节（基于注解的容器配置）演示了如何通过源代码级注解提供大量配置元数据。然而即便在这些示例中，"基础"Bean定义仍需在XML文件中显式定义，注解仅驱动依赖注入机制。

本节描述了一种通过扫描类路径隐式检测候选组件的选项。候选组件是指符合过滤条件且在容器中注册了对应 Bean 定义的类。这消除了使用 XML 进行 Bean 注册的需求。取而代之的是，您可以使用注解（例如 `@Component` ）、 `AspectJ` 类型表达式或自定义过滤条件来选择哪些类在容器中注册了 Bean 定义。

可以使用Java而非XML文件来定义Bean: `@Configuration` 、 `@Bean` 、 `@Import` 和 `@DependsOn` 注解。

## @Component 和衍生注解
Spring 提供了更多类型的注解：`@Component` 、 `@Service` 和 `@Controller` 。 `@Component` 是适用于任何 Spring 管理组件的通用类型。 `@Repository` 、 `@Service` 和 `@Controller` 则是 `@Component` 的特化形式，分别在持久层、服务层和展示层。因此，虽然可以使用 `@Component` 标注组件类，但若改用 `@Repository` 、 `@Service` 或 `@Controller` 标注，这些类将更适于工具处理或与切面关联。例如，这些注解是切入点的理想目标。在 Spring 框架的未来版本中， `@Repository` 、 `@Service`  和 `@Controller` 可能承载更多语义。因此，若需在服务层选择 `@Component` 或 `@Service` ， `@Service` 显然是更优选项。同理， `@Repository` 注解用于标记任何实现存储库（也称为数据访问对象或DAO）角色或模式的类。该标记注解的功能之一是自动转换异常。

## 
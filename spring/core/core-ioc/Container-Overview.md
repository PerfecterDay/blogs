# Container Overview

`org.springframework.beans` 和 `org.springframework.context` 包构成了 Spring 框架 IoC 容器的基础。BeanFactory 接口提供了一种高级配置机制，能够管理任何类型的对象。ApplicationContext 是 BeanFactory 的子接口，它增加了：
+ 更轻松地集成 Spring 的 AOP 功能
+ Message resource 处理（用于国际化）
+ 事件发布于处理
+ 应用层特有的上下文，例如用于Web应用程序的WebApplicationContext。

简而言之， `BeanFactory` 提供了配置框架和基本功能，而 `ApplicationContext` 则增加了更多企业级特定功能。 `ApplicationContext` 是 `BeanFactory` 的完整超集。

在Spring中，构成应用程序核心并由Spring IoC容器管理的对象称为Bean。Bean是由Spring IoC容器实例化、组装和管理的对象。Bean及其间的依赖关系，都体现在容器使用的配置元数据中。


`org.springframework.context.ApplicationContext` 接口代表 Spring IoC 容器，负责实例化、配置和组装 Bean。容器通过读取配置元数据来获取关于实例化、配置和组装组件的指令。配置元数据可以是带注解的组件类、包含工厂方法的配置类，或是外部XML文件或Groovy脚本。无论采用何种格式，您都可据此构建应用程序，并实现组件间的丰富交互依赖关系。

Spring 核心中包含了多个实现 ApplicationContext 接口的类。在独立应用程序中，通常会创建 AnnotationConfigApplicationContext 或 ClassPathXmlApplicationContext 的实例。

下图展示了Spring工作原理的高级视图。您的应用程序类与配置元数据相结合，因此在ApplicationContext创建并初始化后，您将获得一个完全配置且可执行的系统或应用程序。

container-magic.png

## Configuration Metadata
如前图所示，Spring IoC容器会消耗一种配置元数据。这种配置元数据代表了您作为应用程序开发人员如何指示Spring容器对应用程序中的组件进行实例化、配置和组装。

Spring IoC容器本身与配置元数据的实际写入格式完全解耦。如今，许多开发者为其Spring应用程序选择基于Java的配置：
+ 注解式配置：在应用程序的组件类上使用注解式配置元数据来定义Bean。
+ 基于Java的配置：通过基于Java的配置类在应用程序类外部定义Bean。要使用这些功能，请参阅@Configuration、@Bean、@Import和@DependsOn注解。

These bean definitions correspond to the actual objects that make up your application. Typically, you define service layer objects, persistence layer objects such as repositories or data access objects (DAOs), presentation objects such as Web controllers, infrastructure objects such as a JPA EntityManagerFactory, JMS queues, and so forth. Typically, one does not configure fine-grained domain objects in the container, because it is usually the responsibility of repositories and business logic to create and load domain objects.


这些bean定义对应于构成应用程序的实际对象。通常，您需要定义服务层对象、持久层对象（如存储库或数据访问对象DAO）、展示层对象（如Web控制器）、基础架构对象（如JPA实体管理器工厂、JMS队列等）。通常情况下，细粒度的领域对象无需在容器中进行配置，因为创建和加载领域对象通常是存储库和业务逻辑的职责。


## Using the Container
```
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");

GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

随后可通过 `getBean` 方法获取 Bean 实例。ApplicationContext 接口还提供其他获取 Bean 的方法，但理想情况下应用程序代码应完全避免使用这些方法。实际上，应用程序代码不应包含任何 `getBean()` 调用，从而彻底消除对 Spring API 的依赖。例如，Spring 与 Web 框架的集成可为控制器、JSF 管理 Bean 等组件提供依赖注入，允许您通过元数据（如自动装配注解）声明对特定 Bean 的依赖关系。
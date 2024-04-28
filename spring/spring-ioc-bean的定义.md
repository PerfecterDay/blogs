# Spring IoC Bean的定义
{docsify-updated}


## Bean 的 scope
Spring容器最初提供了两种bean的scope类型：`singleton` 和 `prototype` ，但发布2.0之后，又引入了另外三种scope类型，即 `request` 、 `session` 和 `global session` 类型。不过这三种类型有所限制，只能在Web应用中使用。
+ `singleton`: 标记为拥有singleton scope的对象定义，在Spring的IoC容器中只存在一个实例，所有对该对象的引用将共享这个实例。该实例从容器启动，并因为第一次被请求而初始化之后，将一直存活到容器退出，也就是说，它与IoC容器“几乎”拥有相同的“寿命”。
+ `prototype` ：容器在接到该类型对象的请求的时候，会每次都重新生成一个新的对象实例给请求方。虽然这种类型的对象的实例化以及属性设置等工作都是由容器负责的，但是只要准备完毕，并且对象实例返回给请求方之后，容器就不再拥有当前返回对象的引用，请求方需要自己负责当前返回对象的后继生命周期的管理工作，包括该对象的销毁。也就是说，容器每次返回给请求方一个新的对象实例之后，就任由这个对象实例“自生自灭”了。
+ `request`: Spring容器的 XmlWebApplicationContext 会为每个HTTP请求创建一个全新的bean对象供当前请求使用，当请求结束后，该对象实例的生命周期即告结束。
+ `session` ：Spring容器会为每个独立的session创建属于它们自己的全新的bean对象实例。与request相比，除了可能更长的存活时间，其他方面真是没什么差别。
+ `global session`: 只有应用在基于portlet的Web应用程序中才有意义，它映射到portlet的global范围的 session。如果在普通的基于servlet的Web应用中使用了这个类型的scope，容器会将其作为普通的session类型的scope对待。


### classpath和classpath*的区别

记录一下踩坑：
有两个项目A和B，A依赖B，最终运行的是A。使用mybatis做数据库访问，在一个B的jar包中写好了mapper.xml文件，A项目运行时，能运行A中的mapper文件，找不到B中绑定的sql语句。就是说mybatis扫描mapper.xml文件时，能扫描到项目本身的mapper，却不能扫描到依赖jar包中的mapper，仔细查看文件才发现配置文件有问题：

    mapper-locations: classpath:mapper/**/*.xml
改为

    mapper-locations: classpath*:mapper/**/*.xml
后，问题解决。

classpath和classpath*区别： 
1. classpath：只会到你的class路径中查找找文件。
2. classpath*：不仅包含class路径，还包括jar文件中（class路径）进行查找。

注意： 用classpath*:需要遍历所有的classpath，所以加载速度是很慢的；因此，在规划的时候，应该尽可能规划好资源文件所在的路径，尽量避免使用classpath*。



## Spring Framework 的配置方式
Spring Framework 的配置大致可以分为三种方式：

1. 创建 XML 配置
这是最传统的配置方式，使用 &lt;beans&gt; XML 命名空间，将需要注入的 bean 配置在 ,&lt;bean&gt; 标签下即可配置 bean 。

2. 创建混合配置
XML 配置文件的缺点是太繁杂，一个大型的企业级应用中，可能会定义数百个 bean ,每个 bean 都至少要三行代码的话，光是配置文件都要数千行。

spring 注解配置的核心在于 **组件扫描** 和 **注解配置**。 
#### 组件扫描的开启
XML 文件中使用 `<context:annotation-config>` 和 `<context:component-scan>` 元素即可开启**组件扫描** 和 **注解配置** 。不过`<context:component-scan>` 也有 `<context:annotation-config>` 注解的功能，因此只要配置 `<context:component-scan>` 就能开启上述功能

#### 扫描注解配置
通过使用注解扫描， Spring 将扫描通过特定注解指定的包去扫描类。扫描路径下的所有标注了 `@Componengt` 注解的类都将变成由 Spring 管理的 Bean，这意味着 Spring 将负责实例化它们并注入它们的依赖对象。

其它符合组件扫描的注解： 所有标注了 `@Component` 的注解都将变成组件扫描注解，任何标注了一个组件扫描注解的注解也将编程组件扫描注解。因此， 标注了 `@Controller` 、 `@Service`  、 `@Repository` 注解的类也将编程Spring 管理的bean。

与组件扫描注解配合使用的另一个注解是 `@Autowired` 。可以为任何公开、保护和私有的字段或接受一个或多个参数的 **设置方法** 标注 `@Autowired`。 `@Autowired` 声明了 Spring 实例化之后应该注入的依赖对象，可以指定当前bean定义采用某种类型的自动绑定模式。这样，你就无需手工明确指定该bean定义相关的依赖关系，从而也可以免去一些手工输入的工作量。并且它也可以用于构造器，通常Spring管理的bean类必须有无参构造函数，但对于只含有一个标注了 `@Autowired` 的构造器的类， Spring 将使用该构造器并注入所有的构造器参数。

Spring提供了5种自动绑定模式，即 `no、byName、byType、 constructor 和 autodetect`:
+ no ： 容器默认的自动绑定模式，也就是不采用任何形式的自动绑定，完全依赖手工明确配置各个bean之间的依赖关系。
+ byName: 按照类中声明的实例变量的名称，与XML配置文件或注解配置中声明的bean定义的beanName的值进行匹配，相匹配的bean定义将被自动绑定到当前实例变量上。
+ byType: 容器会根据当前bean定义类型，分析其相应的依赖对象类型，然后到容器所管理的所有bean定义中寻找与依赖对象类型相同的bean，将找到的符合条件的bean自动绑定到当前bean
+ constructor: 它与byType类型的绑定模式类似。不过，constructor是匹配构造方法的参数类型，而不是实例属性的类型。
+ autodetect： 这种模式是byType和constructor模式的结合体，如果对象拥有默认无参数的构造方法，容器会优先考虑byType的自动绑定模式。否则，会使用constructor模式


任何情况下，如果Spring无法为依赖找到匹配的bean，将抛出异常并启动失败。

同样，如果为依赖找到多个匹配的 bean，它也将抛出异常并启动失败。这种情况下，可以使用 `@Qualifier` 或 `@Primary` 注解解决。 通过 `@Qualifier` 可以使用指定名字的依赖。而使用 `@Primary` 标注的类表示在出现多个符合条件的依赖时，应该优先使用标注了该注解的bean。

### 使用 @Configuration 配置
上述两种配置方式基本上都还是要依赖 XML ，第二种方式中要在 XML 中开启注解组件扫描和启用注解配置功能。不过，在使用 XML 配置Spring时有一些缺点：
1. XML 难于调试。
2. 不能对 XML 配置进行单元测试。因为 XML 配置会启动整个应用程序并加载所有 bean ，实际上不是单元测试而是继承测试。

使用 `AnnotationConfigWebApplicationContext` 启动 Spring ，并调用该类的 `register(Class<T> configClass)` 方法注册配置类即可实现 java 配置。 这些配置类必须标注上 `@Configuration` 注解，也必须有默认构造函数，配置类中标注了 `@Bean` 注解的无参方法将注册 bean。

在配置类中可以使用下列注解代替 XML 中的一些配置元素：
1. `@ComponentScan` : 替代的是 `<context:component-scan>` ，启用组件扫描功能
2. `@EnableAspectAutoProxy` : 替代的是 `<aop:aspect-autoprooxy>` 元素，启用对标注了 `@Aspect` 注解的类的处理，面向切面编程时使用
3. `@EnableAsync` : 替代的是 `<async:*>` 命名空间，启用Spring的异步 `@Async` 方法执行
4. `@EnableCaching` : 替代的是 `<cache:*>` 命名空间。
5. `@EnableScheduling` : 替代的是 `<task:*>` 命名空间，激活标注了 `@Scheduled` 的计划的执行。
6. `@EnableTransactionManagement` : 替代的是 `<tx:annotation-driven>`， 可以为标注了 `@Transactional` 注解的方法启用事务管理。
7. `@EnableWebMvc` : 替代的是 `<mvc:annotation-driven>`， 激活了注解驱动的控制器请求映射。该注解将激活一个非常复杂的配置，通常需要进行自定义。通过使标注了 `@Configuration` 注解的配置里实现 `WebMvcConfigurer` 或者继承 `WebMvcConfigurerAdapter` 来定义整个 Web MVC 配置。

另外，某些情况下，我们不得不使用 XML 配置，比如要注入某个 Jar 包中的 Bean ，无法为该 bean 的类标注上类似 `@Component` 的注解。我们可以使用 `@Import` 和 `@ImportResource` 注解来加载其它配置类或 XML 配置文件。
```
//Java注解的方式配置 Bean
@Configuration
@Import({DatabaseConfiguration.class, ClusterConfiguraton.class})
@ImportResource("classpath:com/wrox/config/spring-security.xml)
@ComponentScan("com.baicy.wang")
public class ExampleConfiguration{
    @Bean
    public Bean getBean(){
        return new Bean();
    }
    .....
}
//使用上述配置启动上下文
AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
context.register(ExampleConfiguration.class);
```
#  Springboot 自动配置原理
{docsify-updated}


## 自动配置原理
Springboot（通常是各种starter） 实际上就是为我们写好了很多 @Configuration 注解的配置类，这些类中大量使用了基于 @Conditional 注解的配置，以在满足一些条件时自动为我们注入一些 Bean 。那么还有一个问题，我们知道，要使 @Configuration 注解的配置类生效，主要有三种方式：
1. 它处于自动扫描的包下，会被自动扫描
2. 被其他配置类用 @Import 引用
3. 采用编程的方式手动注册到应用上下文
   ```
   AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
   context.register(ExampleConfiguration.class);
   ```
这些方式都必须要求使用者（写代码）手写相关的代码才能完成。Springboot 使用了另一种方式来保证了这些预定义的配置类被加载。

Spring Boot 会检查发布的 jar 中是否存在 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件(Springboot 中的`AutoConfigurationMetadata`类实现)。该文件应列出配置类，每行一个类名，如下例所示：
```
com.mycorp.libx.autoconfigure.LibXAutoConfiguration
com.mycorp.libx.autoconfigure.LibXWebAutoConfiguration
```
这些类中往往使用了 `@Conditional` 等注解实现了自定义的自动配置。

自动配置必须通过在导入文件中命名的方式加载。确保它们被定义在特定的包空间中，并且永远不会成为组件扫描的目标。此外，自动配置类不应允许组件扫描查找其他组件,应使用特定的 `@Import` 注解来代替。

### @Configuration 注解
使用 @Configuration 注解的类，可以在其内定义用 @Bean 注解的方法，方法的返回对象将注入到 Spring bean 容器中 。

### @Conditional 注解
`@Conditional` 是最基础的注解，许多其他注解都是扩展自该注解。

```
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {
    Class<? extends Condition>[] value();
}
```
该注解的值是一个继承自 `Condition` 接口的类， `Condition` 接口只有一个返回 `boolean` 的 `matches` 方法：
```
@FunctionalInterface
public interface Condition {
    boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}
```
所以该注解可以和`@Conmponent/@Service/@Repository/@Configuration`等一起标注在一个类上，当Condition中的接口返回 true 时，这些注解才会生效。另外，该注解也可以和 `@Bean` 注解一起使用，以控制一个 bean 只在条件满足的时候才会被注入。

基于 `@Conditional` 注解， springboot 实现了很多扩展的注解，这些注解通常配合 `@Configuration` 注解一起使用以控制某个配置类是否生效，来达到自动配置的效果：
1. `@ConditionalOnClass/@ConditionalOnMissingClass` : 当classpath中存在/不存在某个类时，配置生效
2. `@ConditionalOnMissingBean/@ConditionalOnBean` ：当spring上下文/beanfactory 中缺失/存在某个类型bean 时，配置生效
3. `@ConditionalOnNotWebApplication/@ConditionalOnWebApplication`: 只有时Web 类型应用时配置生效
4. `@ConditionalOnProperty("my.property")`：只有配置项 my.property 存在时，配置才生效
5. `@ConditionalOnResource("classpath:my.properties")`：只有配置文件 my.property 存在时，配置生效
6. `@ConditionalOnJava(JavaVersion.EIGHT)`：只有Java 版本时才生效
7. `@ConditionalOnCloudPlatform(CloudPlatform.Heroku)`：只有在指定的云平台下才生效
8. `@ConditionalOnExpression("someSpELExpression)`：只有在SPEL表达式为true 的情况下生效
9. `@ConditionalOnJndi("java:comp/env/ejb/myEJB")`：只有在指定jndi存在的情况下才生效

更多 `@Conditional` 注解，可参考[官方文档](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration.condition-annotations)

## 自定义自动配置  [官方指导](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration)
自动配置必须通过在导入文件中命名的方式加载。确保它们被定义在特定的包空间中，并且永远不会成为组件扫描的目标。此外，自动配置类不应允许组件扫描查找其他组件。应使用特定的 `@Import` 注解来代替。

另外，自动配置类必须啊放在一个单独的jar 中。

1. 新建一个自动配置类
   ```
    @Configuration
    public class AutoConfigurationTest {

        @Bean
        @ConditionalOnProperty("test")
        public Test test(){
            return new Test();
        }
    }
   ```
2. resources 目录下新建 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件并写入以下内容。
   `com.gtja.test.AutoConfigurationTest`

## 自定义starter [官方指导](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration.custom-starter)
Spring Boot 2.7 引入了用于注册自动配置的新 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件，同时保持了与 spring.factories 中注册的向后兼容性。在此版本中，使用 `org.springframework.boot.autoconfigure.EnableAutoConfiguration` 关键字在 `spring.factories` 中注册自动配置的支持已被移除，转而使用 imports 文件。`spring.factories` 中其他键下的条目不受影响。

### @SpringBootApplication 注解
通常我们使用 @SpringBootApplication 在启动 springboot 的 main 方法所在类上标注来启动自动注解功能。

`@SpringBootApplication` 实际上是三个注解的合集：
```
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication{...}
```

1. @SpringBootConfiguration  

    `@SpringBootConfiguration` 是一个类级注解，是 Spring Boot 框架的一部分。它表示一个类提供了应用程序配置。
    `@SpringBootConfiguration` 是 `@Configuration` 注解的替代品。其主要区别在于 `@SpringBootConfiguration` 允许自动定位配置。这对单元测试或集成测试特别有用。
2. @EnableAutoConfiguration
  
   `@EnableAutoConfiguration` 注解使 Spring Boot 能够自动配置应用程序上下文。因此，它会根据类路径中包含的 jar 文件和我们定义的 Bean 自动创建和注册 Bean。
3. @ComponentScan

   @ComponentScan 使 Spring 能够扫描配置、控制器、服务和我们定义的其他组件。可以从指定的包开始扫描，使用 `basePackageClasses()` 或 `basePackages()` 来定义指定的包。如果没有指定包，Spring 会将声明 `@ComponentScan` 注解的类的所在的包作为起始包，扫描该包及其子包下的所有配置。
## Springboot 自动配置原理
{docisify-updated}

- [Springboot 自动配置原理](#springboot-自动配置原理)
	- [@Configuration 注解](#configuration-注解)
	- [@Conditional 注解](#conditional-注解)
	- [@EnableAutoConfiguration/@SpringBootApplication](#enableautoconfigurationspringbootapplication)
	- [Springboot 自动配置的本质](#springboot-自动配置的本质)
### @Configuration 注解
使用 @Configuration 注解的类可以在其中用 @Bean 注解的方法为 Spring 注入Bean。


### @Conditional 注解
@Conditional 是最基础的注解，许多其他注解都是扩展自该注解。

```
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {
    Class<? extends Condition>[] value();
}
```
该注解的值是一个继承自 Condition 接口的类， Condition 接口只有一个返回 boolean 的 matches 方法：
```
@FunctionalInterface
public interface Condition {
    boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}
```
所以该注解可以和@Conmponent/@Service/@Repository/@Configuration等一起标注在一个类上，当Condition中的接口返回 true 时，这些注解才会生效。另外，该注解也可以和 @Bean 注解一起使用，以控制一个 bean 只在条件满足的时候才会被注入。

基于 @Conditional 注解， springboot 实现了很多扩展的注解，这些注解通常配合 `@Configuration` 注解一起使用一控制某个配置类是否生效，来达到自动配置的效果：
1. @ConditionalOnClass/@ConditionalOnMissingClass : 当classpath中存在/不存在某个类时，配置生效
2. @ConditionalOnMissingBean/@ConditionalOnBean ：当spring上下文/beanfactory 中缺失/存在某个类型bean 时，配置生效
3. @ConditionalOnNotWebApplication/@ConditionalOnWebApplication
4. @ConditionalOnProperty("my.property")
5. @ConditionalOnResource("classpath:my.properties")
6. @ConditionalOnJava(JavaVersion.EIGHT)
7. @ConditionalOnCloudPlatform(CloudPlatform.Heroku)
8. @ConditionalOnExpression("someSpELExpression)
9. @ConditionalOnJndi("java:comp/env/ejb/myEJB")

### @EnableAutoConfiguration/@SpringBootApplication 
Springboot 使用 @EnableAutoConfiguration 注解来启用自动注解功能，通常我们使用 @SpringBootApplication 在启动 springboot 的 main 方法所在类上标注。

### Springboot 自动配置的本质
Springboot 实际上就是为我们写好了很多 @Configuration 注解的配置类，这些类中大量使用了基于 @Conditional 注解的配置，以在满足一些条件时自动为我们注入一些 Bean 。
那么还有一个问题，我们知道，要使 @Configuration 注解的配置类生效，主要有三种方式：
1. 它处于自动扫描的包下，会被自动扫描
2. 被其他配置类用 @Import 引用
3. 采用编程的方式手动注册到应用上下文
   ```
   AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
   context.register(ExampleConfiguration.class);
   ```
这些方式都必须要求使用者（写代码）手写相关的代码才能完成。Springboot 使用了另一种方式来保证了这些预定义的配置类被加载。

Springboot 在加载时会扫描所有 jar 下 META-INF/spring.factories 文件，然后在这个文件中写上所有需要加载的配置类。
如果我们自己也想编写一个自动配置的 starter， 那么也可以写好配置类后，然后将配置类写入打包 jar 的META-INF/spring.factories 文件中，这样只要别人使用了我们的jar，我们的自动配置类就能神生效。
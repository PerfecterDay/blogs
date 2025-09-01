# Spring 声明式事务
{docsify-updated}

## 声明式事务原理
关于 Spring Framework 的声明式事务支持，需要掌握的最重要的概念是：这种支持是通过 AOP 代理实现的，事务相关的 `advice` 是由元数据（目前基于 XML 或注解）驱动的。AOP 与事务元数据的结合产生了一个 AOP 代理，它使用事务拦截器 `TransactionInterceptor` 与适当的事务管理器 `TransactionManager` 实现来驱动方法调用的增强，从而实现事务管理功能。

Spring Framework 的 `TransactionInterceptor` 为命令式和反应式编程模型提供事务管理。`TransactionInterceptor` 通过检查方法的返回类型来检测所需的事务管理类型。返回 `Publisher` 或 Kotlin `Flow`（或其子类型）等反应式类型的方法使用反应式事务管理 `ReactiveTransactionManager` 。包括 void 在内的所有其他返回类型都使用命令式事务管理 `PlatformTransactionManager` 。

<center><img src="pics/tx.png" alt=""></center>

## @Transactional注解
```
@Transactional
public class DefaultFooService implements FooService {
	@Override
	public Foo getFoo(String fooName) {
		// ...
	}

	@Override
	public void insertFoo(Foo foo) {
		// ...
	}

	@Override
	public void updateFoo(Foo foo) {
		// ...
	}
}
```
如上所述，在类级别使用该注解时，注解表示声明类所有方法（默认是 public/protect 方法）都是事务管理的。`@Transactional` 也可以单独注解每个方法上。如果一个子类继承了一个 `@Transactional` 注解的父类，需要在子类或者其方法中重新加上 `@Transactional` 才能使事务生效。

仅有 `@Transactional` 注解还不足以激活事务行为。 `@Transactional` 注解只是一个元数据，要使 `@Transactional`注解生效，还要在一个 `@Configuration` 配置类中加上 `@EnableTransactionManagement` 。或者在 xml 中使用 `<tx:annotation-driven/>` 。

`@Transactional` 注解最好是注解在具体类的方法，而不是接口中的方法上。由于 Java 注解不能从接口继承，因此在使用 `AspectJ` 模式时， AspectJ 织入器可能无法识别接口声明的注解，导致事务注解被忽略。

### @Transactional的属性设置
```
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {
    @AliasFor("transactionManager")
    String value() default "";

    @AliasFor("value")
    String transactionManager() default "";

    String[] label() default {};

    Propagation propagation() default Propagation.REQUIRED;

    Isolation isolation() default Isolation.DEFAULT;

    int timeout() default -1;

    String timeoutString() default "";

    boolean readOnly() default false;

    Class<? extends Throwable>[] rollbackFor() default {};

    String[] rollbackForClassName() default {};

    Class<? extends Throwable>[] noRollbackFor() default {};

    String[] noRollbackForClassName() default {};
}
```

<center><img src="pics/transactional.png" alt=""></center>

`@Transactional` 注解是一种元数据，用于指定接口、类或方法的具有事务语义。默认的 `@Transactional` 设置如下：
+ 传播设置为 `PROPAGATION_REQUIRED` 
+ 隔离级别为 `ISOLATION_DEFAULT` 
+ readOnly 为 false
+ 事务超时默认为底层事务系统的默认超时，如果不支持超时，则默认为无。
+ 任何 `RuntimeException` 或 `Error` 都会触发回滚，而任何 `checked Exception` 则不会触发回滚。

详细的属性含义见[官方文档](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html#transaction-declarative-attransactional-settings)

### 事务失效
因为事务原理是基于 AOP 代理的，所以只有通过代理引用进入的外部方法调用才会被拦截。这意味着自调用（实际上是目标对象中的一个方法调用目标对象的另一个方法）在运行时不会导致实际事务，即使被调用的方法标记了 `@Transactional` 也是如此。此外，代理必须完全初始化才能正常工作，因此不应在初始化代码中依赖这一功能，例如在 `@PostConstruct` 方法中期望使用spring 事务管理功能。

## 基于XML的声明式事务实例
```xml
<bean id="fooService" class="x.y.service.DefaultFooService"/>

<!-- the transactional advice (what 'happens'; see the <aop:advisor/> bean below) -->
<tx:advice id="txAdvice" transaction-manager="txManager">
	<!-- the transactional semantics... -->
	<tx:attributes>
		<!-- all methods starting with 'get' are read-only -->
		<tx:method name="get*" read-only="true"/>
		<!-- other methods use the default transaction settings (see below) -->
		<tx:method name="*"/>
	</tx:attributes>
</tx:advice>

<!-- ensure that the above transactional advice runs for any execution
	of an operation defined by the FooService interface -->
<aop:config>
	<aop:pointcut id="fooServiceOperation" expression="execution(* x.y.service.FooService.*(..))"/>
	<aop:advisor advice-ref="txAdvice" pointcut-ref="fooServiceOperation"/>
</aop:config>

<!-- don't forget the DataSource -->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
	<property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
	<property name="url" value="jdbc:oracle:thin:@rj-t42:1521:elvis"/>
	<property name="username" value="scott"/>
	<property name="password" value="tiger"/>
</bean>

<!-- similarly, don't forget the TransactionManager -->
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"/>
</bean>
```
该配置用于在根据 `fooService` Bean 定义创建的对象周围创建事务代理proxy。代理通过事务 advice 进行增强配置，这样，当在代理上调用适当的方法时，就会根据与该方法相关的事务配置，启动、暂停或标记为只读等事务。

### 基于XML的事务回滚
在默认配置中，Spring Framework 的事务基础架构代码仅在 `runtime`, `unchecked exceptions` 异常时才会标记事务以进行回滚。也就是说，当抛出的异常是 `RuntimeException` 或者 `Error` 的实例或子类时，会导致事务回滚。

在默认配置中，事务方法抛出的 `checked exceptions` 不会导致回滚。不过可以通过指定回滚规则，准确配置哪些异常类型（包括`checked exceptions`）会导致事务回滚。
```
<tx:advice id="txAdvice" transaction-manager="txManager">
	<tx:attributes>
		<tx:method name="get*" read-only="true" rollback-for="NoProductInStockException"/>
		<tx:method name="updateStock" no-rollback-for="InstrumentNotFoundException"/>
		<tx:method name="*"/>
	</tx:attributes>
</tx:advice>
```

## 事务监听器
我们 `@EventListener` 注解来注册常规事件监听器。如果需要监听事务相关的事件，可以使用 `@TransactionalEventListener` 注解。这样做时，监听器默认会绑定到事务的提交阶段。
```java
@Component
public class MyComponent {

	@TransactionalEventListener
	public void handleOrderCreatedEvent(CreationEvent<Order> creationEvent) {
		// ...
	}
}
```

`@TransactionalEventListener` 注解有一个 `phase` 属性，让你可以自定义监听器应绑定的事务阶段。有效的阶段包括 `BEFORE_COMMIT` 、 `AFTER_COMMIT`（默认）、`AFTER_ROLLBACK` 以及 `AFTER_COMPLETION` ，揽括了事务提交的所有情况（无论是提交还是回滚）。

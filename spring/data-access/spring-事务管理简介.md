# Spring 事务管理简介
{docsify-updated}

全面的事务支持是使用 Spring 框架的最重要原因之一。Spring 框架为事务管理提供了一致的抽象，可带来以下优势：
+ 跨不同事务 API（如 Java Transaction API (JTA)、JDBC、Hibernate 和 Java Persistence API (JPA)) 的一致编程模型。
+ 支持声明式事务管理。
+ 与复杂的事务 API（如 JTA）相比，程序化事务管理 API 更简单。
+ 与 Spring 的数据访问抽象出色集成。

## 全局事务与本地事务
传统上，EE 应用程序开发人员在事务管理方面有两种选择：全局事务或本地事务，这两种选择都有很大的局限性。

### 全局事务
全局事务可让您使用多个事务资源，通常是关系数据库和消息队列。应用服务器通过 JTA 管理全局事务，这是一个繁琐的 API（部分原因是其异常模型）。此外，JTA 的 `UserTransaction` 通常需要从 JNDI 获取，这意味着要使用 JTA 还需要使用 JNDI。全局事务的使用限制了应用程序代码的潜在重用，因为 JTA 通常只能在应用服务器环境中使用。

以前，使用全局事务的首选方法是通过 EJB CMT（容器管理事务）。CMT 是一种声明式事务管理（有别于编程式事务管理）。尽管使用 EJB 本身就必须使用 JNDI，但 EJB CMT 消除了与事务相关的 JNDI 查找需求。它消除了编写 Java 代码来控制事务的大部分（但不是全部）需求。CMT 的主要缺点是与 JTA 和应用服务器环境相关联。此外，只有选择在 EJB（或至少在事务 EJB 门面后面）中实现业务逻辑时，才能使用 CMT。总的来说，EJB 的缺点太多了，所以这不是一个有吸引力的提议，尤其是在声明式事务管理有其他令人信服的替代方案的情况下。

### 本地事务
本地事务是特定于资源的，例如与 JDBC 连接相关的事务。本地事务可能更容易使用，但也有很大的缺点：**它们不能跨多个事务资源运行**。例如，使用 JDBC 连接管理事务的代码无法在全局 JTA 事务中运行。由于应用服务器不参与事务管理，因此无法帮助确保跨多个资源的正确性。(值得注意的是，大多数应用程序只使用一个事务资源）。另一个缺点是本地事务对编程模型具有侵入性。

### Spring 的事务支持
Spring 解决了全局和本地事务的缺点。它能让应用程序开发人员在任何环境中使用一致的编程模型。你只需编写一次代码，就能在不同环境中受益于不同的事务管理策略。Spring Framework 提供声明式和编程式事务管理。大多数用户更喜欢声明式事务管理，我们在大多数情况下都推荐使用声明式事务管理。

在编程式事务管理中，开发人员使用 Spring Framework 事务抽象，它可以在任何底层事务基础架构上运行。在首选的声明式模型中，开发人员通常只编写很少或根本不编写与事务管理相关的代码，因此不依赖 Spring Framework 事务 API 或任何其他事务 API。


## Spring 事务抽象
Spring 事务抽象的关键在于事务策略的概念。事务策略由事务管理器（ `TransactionManager` ）定义，特别是用于命令式事务管理的 `org.springframework.transaction.PlatformTransactionManager` 接口和用于反应式事务管理的 `org.springframework.transaction.ReactiveTransactionManager` 接口。以下列表显示了 `PlatformTransactionManager` API 的定义：
```java
public interface PlatformTransactionManager extends TransactionManager {
	TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;
	void commit(TransactionStatus status) throws TransactionException;
	void rollback(TransactionStatus status) throws TransactionException;
}
```
`TransactionManager` 及其子接口主要基于 SPI 模式工作，也就是说各个服务供应商通过 SPI 机制提供其实现，不过也可以在应用程序代码中以编程方式使用它。由于 `PlatformTransactionManager` 是一个接口，因此可以根据需要轻松地对其进行模拟或存根处理。它与查找策略（如 JNDI）无关。 `PlatformTransactionManager` 的实现与 Spring Framework IoC 容器中的其他对象（或 Bean）一样定义。仅这一点就足以让 Spring Framework 事务成为一个值得使用的抽象，即使在使用 JTA 时也是如此。与直接使用 JTA 相比，可以更轻松地测试事务代码。

### TransactionDefinition
`TransactionDefinition` 接口规定：
+ 传播机制：通常，事务范围内的所有代码都在该事务中运行。不过也可以指定当事务上下文已经存在时运行事务方法的行为。例如，代码可以继续在现有事务中运行（常见情况），也可以暂停现有事务并创建新事务。Spring 提供 EJB CMT 中熟悉的所有事务传播选项。要了解 Spring 中事务传播的语义，请参阅事务传播。
+ [隔离级别](/sql/数据库事务.md#事务的隔离级别)：该事务与其他事务工作隔离的程度。
+ 超时：该事务运行多长时间后会超时并被底层事务基础架构自动回滚。
+ 只读状态：当代码读取但不修改数据时，可以使用只读事务。在某些情况下，只读事务是一种非常有用的优化方式，例如在使用 Hibernate 时。

这些设置反映了标准事务概念。如有必要，请参考讨论事务隔离级别和其他核心事务概念的资源。了解这些概念对于使用 Spring 框架或任何事务管理解决方案都至关重要。

#### 事务传播机制
事务传播行为在 Spring 中由枚举 `Propagation` 定义：
```
public enum Propagation {
    REQUIRED(0),
    SUPPORTS(1),
    MANDATORY(2),
    REQUIRES_NEW(3),
    NOT_SUPPORTED(4),
    NEVER(5),
    NESTED(6);
}
```

##### PROPAGATION_REQUIRED
<center><img src="pics/tx_prop_required.png" width="40%"></center>

当传播设置为 `PROPAGATION_REQUIRED` 时，将为应用该设置的每个方法创建一个逻辑事务作用域。每个逻辑事务作用域都可以单独确定只回滚状态，外部事务作用域在逻辑上独立于内部事务作用域。在标准 `PROPAGATION_REQUIRED` 行为中，所有这些作用域都映射到同一个物理事务。因此，在内部事务作用域中设置的只回滚标记确实会影响外部事务实际提交的机会。

但是，在内部事务作用域设置了只回滚标记的情况下，外部事务本身并没有决定回滚，因此回滚（由内部事务作用域悄悄触发）是意外的。此时会抛出相应的 `UnexpectedRollbackException` 。这是我们所期望的行为，这样事务调用者就不会被误导，以为已经执行了提交，而实际上并没有。因此，如果内部事务（外部调用者不知道）默默地将事务标记为仅回滚，外部调用者仍会调用提交。外层调用者需要收到一个 `UnexpectedRollbackException` 异常，以明确表示执行了回滚。

##### PROPAGATION_REQUIRES_NEW
<center><img src="pics/tx_prop_requires_new.png" width="40%"></center>

与 `PROPAGATION_REQUIRED` 相反， `PROPAGATION_REQUIRES_NEW` 总是为每个受影响的事务范围使用独立的物理事务，从不参与外层范围的现有事务。在这种安排下，底层资源事务是不同的，因此可以独立提交或回滚，外部事务不受内部事务回滚状态的影响，内部事务的锁在完成后立即释放。这种独立的内部事务还可以声明自己的隔离级别、超时和只读设置，而不会继承外部事务的特性。

当内部事务获取自己的资源（如新的数据库连接）时，连接到外部事务的资源将保持绑定。这可能会导致连接池耗尽，如果多个线程有一个活动的外部事务，并等待为其内部事务获取一个新连接，而连接池无法再分配任何此类内部连接，则可能导致死锁。请勿使用 `PROPAGATION_REQUIRES_NEW` ，除非您的连接池大小合适，超过并发线程数至少 1 个。

##### PROPAGATION_NESTED
PROPAGATION_NESTED 使用的是单个物理事务，它可以回滚到多个保存点。这种部分回滚允许内部事务作用域触发其作用域的回滚，尽管某些操作已经回滚，外部事务仍能继续物理事务。此设置通常映射到 JDBC 保存点，因此只适用于 JDBC 资源事务。

### TransactionStatus
`TransactionStatus` 接口为事务代码提供了一种控制事务执行和查询事务状态的简单方法。这些概念应该很熟悉，因为它们是所有事务 API 的共同点。下面的列表显示了 TransactionStatus 接口：
```
public interface TransactionStatus extends TransactionExecution, SavepointManager, Flushable {
	@Override
	boolean isNewTransaction();

	boolean hasSavepoint();

	@Override
	void setRollbackOnly();

	@Override
	boolean isRollbackOnly();

	void flush();

	@Override
	boolean isCompleted();
}
```

### 定义 TransactionManager
无论在 Spring 中选择声明式还是编程式事务管理，定义正确的 `TransactionManager` 实现都是绝对必要的。您通常通过依赖注入来定义该实现。 `TransactionManager` 实现通常需要了解其工作环境：JDBC、JTA、Hibernate 等。以下示例展示了如何定义本地 `PlatformTransactionManager` 实现（在本例中，使用的是普通 JDBC。）

#### JDBC
```xml
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"/>
</bean>
```

#### JTA
```xml
<jee:jndi-lookup id="dataSource" jndi-name="jdbc/jpetstore"/>
<bean id="txManager" class="org.springframework.transaction.jta.JtaTransactionManager" />
```

### Hibernate
也可以轻松使用 Hibernate 本地事务。需要定义一个 Hibernate `LocalSessionFactoryBean` ，应用程序代码可以用它来获取 Hibernate `Session` 实例。
```xml
<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
	<property name="dataSource" ref="dataSource"/>
	<property name="mappingResources">
		<list>
			<value>org/springframework/samples/petclinic/hibernate/petclinic.hbm.xml</value>
		</list>
	</property>
	<property name="hibernateProperties">
		<value>
			hibernate.dialect=${hibernate.dialect}
		</value>
	</property>
</bean>

<bean id="txManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
	<property name="sessionFactory" ref="sessionFactory"/>
</bean>
```

如果使用 Hibernate 和 Jakarta EE 容器管理的 JTA 事务，则应使用 `JtaTransactionManager` 。此外，建议 Hibernate 通过其事务协调器以及可能的连接释放模式配置来感知 JTA：
```xml
<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
	<property name="dataSource" ref="dataSource"/>
	<property name="mappingResources">
		<list>
			<value>org/springframework/samples/petclinic/hibernate/petclinic.hbm.xml</value>
		</list>
	</property>
	<property name="hibernateProperties">
		<value>
			hibernate.dialect=${hibernate.dialect}
		</value>
	</property>
	<property name="jtaTransactionManager" ref="txManager"/>
</bean>

<bean id="txManager" class="org.springframework.transaction.jta.JtaTransactionManager"/>
```

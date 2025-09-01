# spring 数据库连接管理
{docsify-updated}

## DataSource
Spring 通过 `DataSource` 获取与数据库的连接。 `DataSource` 是 JDBC 规范的一部分，是一种通用的连接工厂。它可以让容器或框架从应用代码中隐藏连接池和事务管理问题。使开发人员无需了解如何连接数据库的细节。这是设置数据源的管理员的职责。在开发和测试代码的过程中，开发人员很可能同时扮演这两个角色，但不一定要知道生产数据源是如何配置的。

使用 Spring 的 JDBC 层时，可以从 `JNDI` 获取数据源，也可以使用第三方提供的连接池实现配置自己的数据源。传统的选择有 `Apache Commons DBCP` 和 `C3P0` （带有 bean 风格的数据源类）；对于现代 JDBC 连接池，可以考虑使用 `HikariCP` 。

### DBCP
```
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
	<property name="driverClassName" value="${jdbc.driverClassName}"/>
	<property name="url" value="${jdbc.url}"/>
	<property name="username" value="${jdbc.username}"/>
	<property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="jdbc.properties"/>
```

### C3P0
```
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
	<property name="driverClass" value="${jdbc.driverClassName}"/>
	<property name="jdbcUrl" value="${jdbc.url}"/>
	<property name="user" value="${jdbc.username}"/>
	<property name="password" value="${jdbc.password}"/>
</bean>

<context:property-placeholder location="jdbc.properties"/>
```

### JNDI
如果应用配置在高性能的应用服务器（如WebLogic或WebSphere等）上，则可能更希望使用应用服务器本身提供的数据源。应用服务器的数据源使用 `JNDI` 开放调用者使用，Spring为此专门提供了引用JNDI数据源的 `JndiObjectFactoryBean` 类。下面是一个简单的配置：
```
<bean id="dataSource" class="org.springframework.jndi.JndiobjectFactoryBean"
p:jndiName="java:comp/env/jdbc/bbt"/>
```

## 使用内置数据库支持
`org.springframework.jdbc.datasource.embedded` 包为嵌入式 Java 数据库引擎提供支持。原生支持 HSQL、H2 和 Derby。还可以使用可扩展的 API 来插入新的嵌入式数据库类型和 DataSource 实现。

```
@Configuration
public class DataSourceConfig {

	@Bean
	public DataSource dataSource() {
		return new EmbeddedDatabaseBuilder()
				.generateUniqueName(true)
				.setType(EmbeddedDatabaseType.HSQL) //可选值还有EmbeddedDatabaseType.H2和EmbeddedDatabaseType.DERBY
				.setScriptEncoding("UTF-8")
				.ignoreFailedDrops(true)
				.addScript("schema.sql") //建表语句
				.addScripts("user_data.sql", "country_data.sql") //初始化数据库表语句
				.build();
	}
}
```

## Spring boot 中的 DataSource 配置
`DataSourceAutoConfiguration` 是 springboot 自动配置数据源的配置类。

### HikariCP
[Hikari 连接池](https://github.com/brettwooldridge/HikariCP#gear-configuration-knobs-baby)：

```
spring:
  datasource:
   url: "jdbc:mysql://localhost/test"
    username: "dbuser"
    password: "dbpass"
    hikari:
      autoCommit: true 
      connectionTimeout: 20000
   maximumPoolSize : 30
```

- `autoCommit` : 此属性控制从池中返回的连接的默认自动提交行为。它是一个布尔值。默认值：true

- `connectionTimeout` : 此属性控制客户端（也就是你）从池中等待连接的最大毫秒数。如果超过这个时间而没有连接可用，将抛出一个SQLException。可接受的最低连接超时为250ms。默认值：30000 (30秒)
- `idleTimeout` : 此属性控制了允许一个连接在池中闲置的最大时间。这个设置只适用于`minimumIdle`被定义为小于`maximumPoolSize`的情况。一旦池中的连接达到最小闲置时间，闲置的连接将不会被清退。一个连接是否被清退为空闲连接，最大变化是+30秒，平均变化是+15秒。在这个超时之前，一个连接永远不会被作为空闲退役。值为0意味着空闲的连接永远不会从池中移除。允许的最小值是10000ms（10秒）。默认值：600000（10分钟）
- `keepaliveTime` : 此属性控制HikariCP尝试保持一个连接的频率，以防止它被数据库或网络基础设施超时。这个值必须小于`maxLifetime`值。Keepalive "只会发生在空闲的连接上。当对一个给定的连接进行 "keepalive "的时间到了，该连接将从池中移除，"ping"，然后返回到池中。ping "是以下两种情况之一：调用JDBC4 isValid()方法，或者执行connectionTestQuery。通常情况下，池外的持续时间应该以个位数毫秒甚至亚毫秒来衡量，因此应该很少或没有明显的性能影响。允许的最小值是30000ms（30秒），但最理想的值是在分钟范围内。默认值：0（禁用）
- `maxLifetime` : 此属性控制池中连接的最大寿命。一个正在使用的连接将永远不会被淘汰，只有当它被关闭时才会被删除。在每个连接的基础上，轻微的负衰减被应用，以避免池中的大规模灭绝。我们强烈建议设置这个值，它应该比任何数据库或基础设施施加的连接时间限制短几秒。值为0表示没有最大的寿命（无限的寿命），当然要根据`idleTimeout`的设置。允许的最小值是30000ms（30秒）。默认值：1800000（30分钟）
- `connectionTestQuery` : 如果你的驱动程序支持JDBC4，我们强烈建议不要设置这个属性。这是针对不支持JDBC4 Connection.isValid() API的 "传统 "驱动程序。这是一个查询，在一个连接从池子里给你之前会被执行，以验证与数据库的连接是否仍然有效。同样，尝试在没有这个属性的情况下运行数据库池，如果你的驱动不符合JDBC4标准，HikariCP会记录一个错误，让你知道。默认：无
- `minimumIdle` : 这个属性控制了HikariCP试图在池中保持的最小空闲连接数。如果空闲连接数低于这个值，并且池中的总连接数小于 `maximumPoolSize` ，HikariCP将尽最大努力快速有效地添加额外的连接。然而，为了获得最大的性能和对尖峰需求的响应，我们建议不要设置这个值，而是让HikariCP作为一个固定大小的连接池。默认值：与`maximumPoolSize`相同
- `maximumPoolSize` : 此属性控制了允许池子达到的最大尺寸，包括空闲和使用中的连接。基本上这个值会决定到数据库后端的实际连接的最大数量。这方面的合理值最好由你的执行环境决定。当数据库池达到这个大小，并且没有空闲连接可用时，对getConnection()的调用将在超时前阻塞多达`connectionTimeout`毫秒。请阅读关于池的大小。默认值：10

自动配置的相关类 `DataSourceAutoConfiguration` 、 `DataSourceConfiguration` 。
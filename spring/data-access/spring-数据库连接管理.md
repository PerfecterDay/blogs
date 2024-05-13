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
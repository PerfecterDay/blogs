# Springboot 集成 flyway
{docsify-updated}

> https://docs.spring.io/spring-boot/how-to/data-initialization.html#howto.data-initialization.migration-tool

mysql 从 8.0.31 开始，artifactId 已统一为 `mysql-connector-j` ，不是以前的 `mysql-connector-java` 。
```
<dependency>
  <groupId>com.mysql</groupId>
  <artifactId>mysql-connector-j</artifactId>
  <version>9.0.0</version>
</dependency>
```

Spring Boot 4.0.x 中， `Flyway` 已经不再由 `spring-boot-autoconfigure` 提供自动配置了，从 Spring Boot 4.0 开始，大量可选第三方集成被模块化拆分，数据库迁移工具（Flyway / Liquibase）被移出 `core autoconfigure` 。 所以需要手动添加依赖：
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-flyway</artifactId>
</dependency>

<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-mysql</artifactId>
</dependency>
```

`application.yml` 配置:
```
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/cap
    username: root
    password: 123456

  flyway:
    enabled: true
    locations: classpath:db/migration
```

在 `resources` 文件夹下创建以下文件
```

└── db
    └── migration
        └── V1__create_user_table.sql
```



## 问题
1. mysql 版本过高，返回不支持Mysql 9.5 ,异常栈
```
Caused by: org.flywaydb.core.api.FlywayException: Unsupported Database: MySQL 9.5
	at org.flywaydb.core.internal.database.DatabaseTypeRegister.lambda$getDatabaseTypeForConnection$7(DatabaseTypeRegister.java:142) ~[flyway-core-11.14.1.jar:na]
	at java.base/java.util.Optional.orElseThrow(Optional.java:403) ~[na:na]
	at org.flywaydb.core.internal.database.DatabaseTypeRegister.getDatabaseTypeForConnection(DatabaseTypeRegister.java:142) ~[flyway-core-11.14.1.jar:na]
	at org.flywaydb.core.internal.jdbc.JdbcConnectionFactory.<init>(JdbcConnectionFactory.java:77) ~[flyway-core-11.14.1.jar:na]
    ....
```

解决方案，添加以下依赖告知 flyway 使用的 mysql ，否则默认只有 H2/SqlLite/TestContainers 的支持：
```
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-mysql</artifactId>
</dependency>
```

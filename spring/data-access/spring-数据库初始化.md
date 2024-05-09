# 数据库初始化
> https://docs.spring.io/spring-boot/how-to/data-initialization.html

```
<jdbc:initialize-database data-source="dataSource"
	enabled="#{systemProperties.INITIALIZE_DATABASE}"> 
	<jdbc:script location="..."/>
</jdbc:initialize-database>
````


```
spring:
  sql:
    init:
      continue-on-error: true
      mode: always
      schema-locations: classpath*:db-init/schema.sql
```
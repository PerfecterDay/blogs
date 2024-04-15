#  Redission
{docsify-updated}

Redisson是一个具有内存数据网格功能的Redis Java客户端。它提供了更方便和最简单的方式来处理Redis。Redisson对象提供了一个分离的关注点，这使你能够保持对数据建模和应用逻辑的关注。


### 集成 SpringBoot
```
<dependency>
	<groupId>org.redisson</groupId>
	<artifactId>redisson-spring-boot-starter</artifactId>
	<version>3.22.1</version>
</dependency>
```
如有必要，将redisson-spring-data模块降级，以支持所需的Spring Boot版本，比如集成 SpringBoot 2.7:
```
<dependency>
	<groupId>org.redisson</groupId>
	<artifactId>redisson-spring-boot-starter</artifactId>
	<version>3.22.1</version>
	<exclusions>
		<exclusion>
			<groupId>org.redisson</groupId>
			<artifactId>redisson-spring-data-31</artifactId>
		</exclusion>
	</exclusions>
</dependency>

<dependency>
	<groupId>org.redisson</groupId>
	<artifactId>redisson-spring-data-27</artifactId>
	<version>3.22.1</version>
</dependency>
```

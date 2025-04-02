#  MybatisPlus 多数据源
{docsify-updated}

> https://baomidou.com


### 多数据源配置
1. 添加依赖
   ```
	<dependency>
		<groupId>com.baomidou</groupId>
		<artifactId>dynamic-datasource-spring-boot-starter</artifactId>
		<version>3.5.2</version>
	</dependency>
   ```
2. 配置数据源
   ```
   spring:
	datasource:
	  dynamic:
		primary: user #设置默认的数据源或者数据源组,默认值即为master
		strict: false
		datasource:
			user:
				url: jdbc:mysql://rm-3nsz59cw245j0v25w.mysql.rds.aliyuncs.com:3306/user?useSSL=false&useUnicode=true&characterEncoding=UTF-8
				username: mobileapp_svc
				password: p0BWm71pakWIZ3pH
				driver-class-name: com.mysql.cj.jdbc.Driver
			mmf:
				url: jdbc:mysql://rm-3nsz59cw245j0v25w.mysql.rds.aliyuncs.com:3306/mmf?useSSL=false&useUnicode=true&characterEncoding=UTF-8
				username: mobileapp_svc
				password: p0BWm71pakWIZ3pH
				driver-class-name: com.mysql.cj.jdbc.Driver
   ```
3. 使用 `@DS("user")` 注解
	```
	@Mapper
	@DS("user")
	public interface CouponMapper extends BaseMapper<CouponEntity> {
	}
	```
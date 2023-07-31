## Springboot 配置文件加载及常见配置
{docsify-updated}

- [Springboot 配置文件加载及常见配置](#springboot-配置文件加载及常见配置)
	- [加载配置文件及使用](#加载配置文件及使用)
	- [常用配置](#常用配置)


> https://blog.csdn.net/u013217730/article/details/116716418

### 加载配置文件及使用
1. 加载自定义路径下的配置文件  
    Spring/Springboot 中可以使用 `@PropertySource`/`@PropertySources` 注解加载指定路径的配置文件：
    ```
    //动态加载文件，根据 envTarget 的值确定，默认为 persistence-mysql.properties 文件
    @PropertySource({ 
    "classpath:persistence-${envTarget:mysql}.properties"
    })
    public class PropertiesWithJavaConfig {
        //...
    }

    //加载多配置文件
    @PropertySources({
        @PropertySource("classpath:foo.properties"),
        @PropertySource("classpath:bar.properties")
    })
    public class PropertiesWithJavaConfig {
        //...
    }

    // XML 配置文件
    <context:property-placeholder location="classpath:foo.properties, classpath:bar.properties"/>
    ```
2. 使用配置文件  
    1. 可以使用 `@Value` 注解使用配置文件中的配置值。  
        ```
        //读取 jdbc.url 的值，如果没有就为defaultUrl
        @Value( "${jdbc.url:defaultUrl}" )
        private String jdbcUrl;
        ```
		`@Value` 加载配置时，有两种方式：如上所示的`${...}`，还可以使用SPEL（Spring Expression Language）`#{...}`

        XML 配置的情况下:
        ```
        <bean id="dataSource">
            <property name="url" value="${jdbc.url}" />
        </bean>
        ```
    2. java注解方式加载的配置文件中的值会保存到 Spring 内置的 `Environment` 对象中，所以也可以使用下述方式获取配置值  
        ```
        @Autowired
        private Environment env;
        ...
        dataSource.setUrl(env.getProperty("jdbc.url"));
        ```
        注意：property-placeholder 方式加载的配置文件，不会保存到 `Environment` 对象中
    3. 使用分层配置的情况下，可以使用 `@ConfigurationProperties` 来加载  
        ```
        database.url=jdbc:postgresql:/localhost:5432/instance
        database.username=foo
        database.password=bar

        @ConfigurationProperties(prefix = "database")
        public class Database {
            String url;
            String username;
            String password;        
        }
        ```
3. 多环境配置  
    假设我们现在有三套部署环境：
    + Dev
    + Stage
    + Production
    每套环境都有不同的配置存储在不同的配置文件中，但是配置项大都是一样的，配置值不同。在这种情况下，我们就会用到上面的动态配置文件加载：
    ```
    @PropertySource({ 
    "classpath:persistence-${envTarget:mysql}.properties"
    })
    ```
    然后我们再 JVM 命令行中指定 envTarget 的值，不同环境下指定不同的值就能加载指定文件了：
    ```
    -DenvTarget=dev
    -DenvTarget=stage
    ```
4. Springboot中的配置文件  
    上述的配置是Spring和Springboot都支持的，在Spring Boot中，还有更多的配置方式。  
    Springboot 默认会加载 `src/main/resources/application.properties`文件，可以通过命令行参数指定默认配置文件的路径从而加载指定自定义的文件：
    ```
    jvm参数： -Dspring.config.location=classpath:/configuration/stage/app.yaml
    命令行参数：--spring.config.location=classpath:/configuration/stage/app.yaml
    ```
    多环境配置文件名需要满足 application-{profile}.properties 的格式，其中 {profile} 对应你的环境标识，比如：
       1. 开发环境：application-dev.properties  
       2. 测试环境：application-test.properties  
       3. 生产环境：application-prod.properties    
    至于哪个具体的配置文件会被加载，可以在在application.properties文件中或者命令行参数设置`spring.profiles.active`属性值来设置，其值对应{profile}值：
    ```
    spring.profiles.active=stage
    -Dspring.profiles.active=stage
    ```
    注意，即使配置了加载某个 profile 文件，默认的 application.properties 还是会被加载，只不过 profile 配置文件中的配置有更高的优先级。
5. 一个好的倡议
    我们可以在 `src/main/resources/configuration` 目录下新建三个文件夹分别为 dev/stage/prod ,分别对应三种环境，三个文件中配置相同的配置文件。然后删除默认的 application.properties 配置文件，分别在三个文件夹下建立 app.properties 文件，并分别配置上 env=dev/stage/prod 配置项，然后代码中相同的配置文件只要动态加载 ${env} 即可。但是，在启动程序时要指定使用对应环境的 application.properties 配置文件（`--spring.config.location=classpath:/configuration/{stage)/app.properties` ）。

### 常用配置
1. 配置端口： `server.port=9090` 
2. 配置请求路径： `server.servlet.context-path=/trade` 
3. 配置应用名(服务注册时会被注册成服务名)： `spring.application.name=demo`
4. 配置启动初始化 Servlet： `spring.mvc.servlet.load-on-startup=1`
5. 日志设置：
   ```
    logging.path=/user/local/log
    logging.level.com.favorites=DEBUG
    logging.level.org.springframework.web=INFO
    logging.level.org.hibernate=ERROR
    ```
6. 数据库配置：
     ```
    spring.datasource.url=jdbc:mysql://localhost:3306/test
    spring.datasource.username=root
    spring.datasource.password=root
    spring.datasource.driver-class-name=com.mysql.jdbc.Driver
    ```
7. 配置静态文件的URL映射：`spring.mvc.static-path-pattern=/resources/**`
8. 配置静态文件的目录： `spring.resources.static-locations=classpath:/mystatic`
9. 关闭 banner :`spring.main.banner-mode=off`
10. 启用 http2 :`server.http2.enabled=true`

### 数据库连接池配置
`DataSourceProperties` 以了解更多支持的选项。这些是标准的选项，不管实际的实现是什么，都可以工作。也可以通过使用各自的前缀（`spring.datasource.hikari.*`、`spring.datasource.tomcat.*`、`spring.datasource.dbcp2.*`和`spring.datasource.oracleucp.*`）来微调特定实现的设置。更多细节请参见你所使用的连接池实现的文档。

如果你使用Tomcat连接池，你可以定制许多额外的设置，如下面的例子中所示：
```
spring:
  datasource:
    tomcat:
      max-wait: 10000
      max-active: 50
      test-on-borrow: true
```

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
+ `autoCommit` : 此属性控制从池中返回的连接的默认自动提交行为。它是一个布尔值。默认值：true
+ `connectionTimeout` : 此属性控制客户端（也就是你）从池中等待连接的最大毫秒数。如果超过这个时间而没有连接可用，将抛出一个SQLException。可接受的最低连接超时为250ms。默认值：30000 (30秒)
+ `idleTimeout` : 此属性控制了允许一个连接在池中闲置的最大时间。这个设置只适用于`minimumIdle`被定义为小于`maximumPoolSize`的情况。一旦池中的连接达到最小闲置时间，闲置的连接将不会被清退。一个连接是否被清退为空闲连接，最大变化是+30秒，平均变化是+15秒。在这个超时之前，一个连接永远不会被作为空闲退役。值为0意味着空闲的连接永远不会从池中移除。允许的最小值是10000ms（10秒）。默认值：600000（10分钟）
+ `keepaliveTime` : 此属性控制HikariCP尝试保持一个连接的频率，以防止它被数据库或网络基础设施超时。这个值必须小于`maxLifetime`值。Keepalive "只会发生在空闲的连接上。当对一个给定的连接进行 "keepalive "的时间到了，该连接将从池中移除，"ping"，然后返回到池中。ping "是以下两种情况之一：调用JDBC4 isValid()方法，或者执行connectionTestQuery。通常情况下，池外的持续时间应该以个位数毫秒甚至亚毫秒来衡量，因此应该很少或没有明显的性能影响。允许的最小值是30000ms（30秒），但最理想的值是在分钟范围内。默认值：0（禁用）
+ `maxLifetime` : 此属性控制池中连接的最大寿命。一个正在使用的连接将永远不会被淘汰，只有当它被关闭时才会被删除。在每个连接的基础上，轻微的负衰减被应用，以避免池中的大规模灭绝。我们强烈建议设置这个值，它应该比任何数据库或基础设施施加的连接时间限制短几秒。值为0表示没有最大的寿命（无限的寿命），当然要根据`idleTimeout`的设置。允许的最小值是30000ms（30秒）。默认值：1800000（30分钟）
+ `connectionTestQuery` : 如果你的驱动程序支持JDBC4，我们强烈建议不要设置这个属性。这是针对不支持JDBC4 Connection.isValid() API的 "传统 "驱动程序。这是一个查询，在一个连接从池子里给你之前会被执行，以验证与数据库的连接是否仍然有效。同样，尝试在没有这个属性的情况下运行数据库池，如果你的驱动不符合JDBC4标准，HikariCP会记录一个错误，让你知道。默认：无
+ `minimumIdle` : 这个属性控制了HikariCP试图在池中保持的最小空闲连接数。如果空闲连接数低于这个值，并且池中的总连接数小于 `maximumPoolSize` ，HikariCP将尽最大努力快速有效地添加额外的连接。然而，为了获得最大的性能和对尖峰需求的响应，我们建议不要设置这个值，而是让HikariCP作为一个固定大小的连接池。默认值：与`maximumPoolSize`相同
+ `maximumPoolSize` : 此属性控制了允许池子达到的最大尺寸，包括空闲和使用中的连接。基本上这个值会决定到数据库后端的实际连接的最大数量。这方面的合理值最好由你的执行环境决定。当数据库池达到这个大小，并且没有空闲连接可用时，对getConnection()的调用将在超时前阻塞多达`connectionTimeout`毫秒。请阅读关于池的大小。默认值：10

自动配置的相关类 `DataSourceAutoConfiguration` 、 `DataSourceConfiguration` 。

### 配置加密-Vault

安装 vault :
```
brew tap hashicorp/tap
brew install hashicorp/tap/vault
```

启动服务端： `vault server -dev`
Unseal Key: CHisuzHJWLryN/oflCPJPjRTglWj7xUm22NW24xo8oA=
Root Token: hvs.qbUz87luxIjXMeMl83OEwt3c


设置环境变量： 
```
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN="hvs.qbUz87luxIjXMeMl83OEwt3c"
```


`vault status` : 查看服务状态
`vault kv put -mount=secret hello foo=hello bar=world`: 保存一个密钥到 secret/data/hello，key是foo/bar，value是hello/world
`vault kv get -mount=secret hello`: 读取一个 key 的值
`vault kv delete -mount=secret hello` : 删除一个 key
`vault kv undelete -mount=secret -versions=2 hello` : 恢复一个已经删除的key,如果没有永久删除
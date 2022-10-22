# Springboot 配置文件加载及常见配置
{docsify-updated}

> https://blog.csdn.net/u013217730/article/details/116716418

### 加载配置文件及使用
1. 加载自定义路径下的配置文件  
    Spring/SPringboot 中可以使用 `@PropertySource`/`@PropertySources` 注解加载指定路径的配置文件：
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


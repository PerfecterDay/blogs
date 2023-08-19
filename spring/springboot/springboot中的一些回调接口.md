## SpringApplication 中的回调扩展接口
{docsify-updated}
- [SpringApplication 中的回调扩展接口](#springapplication-中的回调扩展接口)
	- [ApplicationContextInitializer](#applicationcontextinitializer)
		- [使用spring.factories方式](#使用springfactories方式)
		- [application.properties添加配置方式](#applicationproperties添加配置方式)
		- [直接通过 `SpringApplication` 的 `add` 方法](#直接通过-springapplication-的-add-方法)
	- [ApplicationRunner 和 CommandLineRunner](#applicationrunner-和-commandlinerunner)
		- [ApplicationRunner](#applicationrunner)
		- [CommandLineRunner](#commandlinerunner)
		- [自定义添加 Runner](#自定义添加-runner)
	- [测试代码](#测试代码)

### ApplicationContextInitializer
回调接口，**用于在刷新 Spring ConfigurableApplicationContext 之前对其进行初始化**。
通常用于需要对应用上下文进行编程初始化的网络应用程序中。例如，针对上下文环境注册属性源或激活配置文件。请参阅 `ContextLoader` 和 `FrameworkServlet` 支持，分别用于声明 "contextInitializerClasses "上下文参数和初始参数。
我们鼓励 `ApplicationContextInitializer` 处理程序检测 Spring 的 Ordered 接口是否已实现或 @Order 注解是否存在，并在调用前对实例进行相应排序。

`ApplicationContextInitializer` 是在springboot启动过程(refresh方法前)调用,主要是在 `ApplicationContextInitializer` 中 `initialize` 方法中拉起了 `ConfigurationClassPostProcessor` 这个类(我在springboot启动流程中有描述)，通过这个 processor 实现了 beandefinition 。言归正传， `ApplicationContextInitializer` 实现主要有3种方式：

#### 使用spring.factories方式
首先我们自定义个类实现了 `ApplicationContextInitializer` ,然后在resource下面新建 `META-INF/spring.factories` 文件。然后在文件中加入：
`org.springframework.context.ApplicationContextInitializer=com.baicy.springbootexplore.initializer.KeyInitializer`

#### application.properties添加配置方式
对于这种方式是通过 `DelegatingApplicationContextInitializer` 这个初始化类中的 `initialize` 方法获取到application.properties中`context.initializer.classes`对应的类并执行对应的initialize方法。只需要将实现了ApplicationContextInitializer的类添加到application.properties即可。
`context.initializer.classes=com.baicy.springbootexplore.initializer.KeyInitializer`
下面是 `DelegatingApplicationContextInitializer` 加载配置文件中 initializer 的代码：
```
private static final String PROPERTY_NAME = "context.initializer.classes";
private List<Class<?>> getInitializerClasses(ConfigurableEnvironment env) {
		String classNames = env.getProperty(PROPERTY_NAME);
		List<Class<?>> classes = new ArrayList<Class<?>>();
		if (StringUtils.hasLength(classNames)) {
			for (String className : StringUtils.tokenizeToStringArray(classNames, ",")) {
				classes.add(getInitializerClass(className));
			}
		}
		return classes;
}
```

#### 直接通过 `SpringApplication` 的 `add` 方法
```
public static void main(String[] args) {
    SpringApplication application = new SpringApplication(SpringbootExploreApplication.class);
    application.addInitializers(new KeyInitializer());
    application.run(args);
}
```


### ApplicationRunner 和 CommandLineRunner
在开发过程中会有这样的场景：需要在容器启动的时候执行一些内容，比如：读取配置文件信息，数据库连接，删除临时文件，清除缓存信息，在Spring框架下是通过 `ApplicationListener` 监听器来实现的。在Spring Boot中给我们提供了两个接口 `CommandLineRunner` 和 `ApplicationRunner` ，来帮助我们实现这样的需求。

在没有指定加载顺序 @Order 时或 @Order 值一致时, 先执行 `ApplicationRunner` 。
如果指定了加载顺序 @Order,则按照 @Order 的顺序进行执行。
说明：数字越小，优先级越高，也就是@Order(1)注解的类会在@Order(2)注解的类之前执行。

Sprinboot 在程序启动之后（ApplicationContext创建、refresh 之后）调用这些接口方法。
<center><img src="pics/springboot-runner.png" width="50%"></center>

#### ApplicationRunner
```
public interface ApplicationRunner {

	/**
	 * Callback used to run the bean.
	 * @param args incoming application arguments
	 * @throws Exception on error
	 */
	void run(ApplicationArguments args) throws Exception;
}
```

#### CommandLineRunner
```
@FunctionalInterface
public interface CommandLineRunner {

	/**
	 * Callback used to run the bean.
	 * @param args incoming main method arguments
	 * @throws Exception on error
	 */
	void run(String... args) throws Exception;

}
```

#### 自定义添加 Runner
不同于 `ApplicationContextInitializer` , `ApplicationRunner` 和 `CommandLineRunner` 是在容器初始化完成、程序启动成功后调用的，Springboot 能够根据类型自动识别并调用，所以要实现自定义的runner 只需要声明一个 bean 并且实现对应接口即可。

### 测试代码
```
@SpringBootApplication(scanBasePackages = {"com.panda.baicy", "com.alibaba.cola"})
public class Application {

    public static void main(String[] args) {
        SpringApplication springApplication = new SpringApplication(Application.class);
        springApplication.addInitializers(applicationContext -> {
            System.out.println(applicationContext.getEnvironment());
        });
        springApplication.run(args);
    }

    @Bean
    public ApplicationRunner applicationRunner(){
        return (ApplicationArguments agrs) ->{
            System.out.println(agrs.getSourceArgs());
        };
    }

    @Bean
    public CommandLineRunner commandLineRunner(){
        return (String[] agrs) ->{
            System.out.println(Arrays.asList(agrs));
        };
    }

}
```
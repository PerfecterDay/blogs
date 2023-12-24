# Springboot 启动过程及扩展点
{docsify-updated}

- [Springboot 启动过程及扩展点](#springboot-启动过程及扩展点)
	- [启动过程](#启动过程)
		- [SpringFactoriesLoader](#springfactoriesloader)
		- [通过 SpringApplication 的 setXXX() 方法自定义 SpringApplication](#通过-springapplication-的-setxxx-方法自定义-springapplication)
	- [扩展点](#扩展点)
		- [SpringApplicationRunListener](#springapplicationrunlistener)
		- [ApplicationContextInitializer](#applicationcontextinitializer)
		- [ApplicationRunner 和 CommandLineRunner](#applicationrunner-和-commandlinerunner)
			- [ApplicationRunner](#applicationrunner)
			- [CommandLineRunner](#commandlinerunner)
			- [自定义添加 Runner](#自定义添加-runner)
		- [测试代码](#测试代码)



## 启动过程
```java
绝大部分Springboot 的启动部分都是:

public static void main(String[] args) throws Exception {
	SpringApplication.run(new Class<?>[0], args);
}

创建一个 SpringApplication 对象并调用它的 run 方法: 
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
	return new SpringApplication(primarySources).run(args);
}

SpringApplication 的构造过程入下：
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
	this.resourceLoader = resourceLoader;
	Assert.notNull(primarySources, "PrimarySources must not be null");
	this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
	this.webApplicationType = WebApplicationType.deduceFromClasspath();
	this.bootstrapRegistryInitializers = new ArrayList<>(
			getSpringFactoriesInstances(BootstrapRegistryInitializer.class));
	setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
	setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
	this.mainApplicationClass = deduceMainApplicationClass();
}

在构造过程中多次用到了 getSpringFactoriesInstances 加载 BootstrapRegistryInitializer/ApplicationContextInitializer/ApplicationListener 等对象，
此处涉及到Springboot 的 SPI 机制：SpringFactoriesLoader ：
private <T> List<T> getSpringFactoriesInstances(Class<T> type, ArgumentResolver argumentResolver) {
	return SpringFactoriesLoader.forDefaultResourceLocation(getClassLoader()).load(type, argumentResolver);
}

public static SpringFactoriesLoader forDefaultResourceLocation(@Nullable ClassLoader classLoader) {
	//public static final String   FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
	return forResourceLocation(FACTORIES_RESOURCE_LOCATION, classLoader);  
}

概括起来说就是 SpringApplication 构造时，会通过 SPI 机制，加载一些类，并设置到自身的相应的属性中。实例化好之后，就会调用 SpringApplication 对象的 run 方法：
public ConfigurableApplicationContext run(String... args) {
	long startTime = System.nanoTime();
	DefaultBootstrapContext bootstrapContext = createBootstrapContext();
	ConfigurableApplicationContext context = null;
	configureHeadlessProperty();
	//获取SpringApplicationRunListener，然后在springboot启动过程中的不同阶段通知（调用）这些listener
	SpringApplicationRunListeners listeners = getRunListeners(args); 
	listeners.starting(bootstrapContext, this.mainApplicationClass);
	try {
		ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
		ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
		Banner printedBanner = printBanner(environment);
		context = createApplicationContext();
		context.setApplicationStartup(this.applicationStartup);
		prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
		refreshContext(context);
		afterRefresh(context, applicationArguments);
		Duration timeTakenToStartup = Duration.ofNanos(System.nanoTime() - startTime);
		if (this.logStartupInfo) {
			new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), timeTakenToStartup);
		}
		listeners.started(context, timeTakenToStartup);
		callRunners(context, applicationArguments); //
	}
	catch (Throwable ex) {
		if (ex instanceof AbandonedRunException) {
			throw ex;
		}
		handleRunFailure(context, ex, listeners);
		throw new IllegalStateException(ex);
	}
	try {
		if (context.isRunning()) {
			Duration timeTakenToReady = Duration.ofNanos(System.nanoTime() - startTime);
			listeners.ready(context, timeTakenToReady);
		}
	}
	catch (Throwable ex) {
		if (ex instanceof AbandonedRunException) {
			throw ex;
		}
		handleRunFailure(context, ex, null);
		throw new IllegalStateException(ex);
	}
	return context;
}
```

### SpringFactoriesLoader 
以下是Springboot源码中对 SpringFactoriesLoader 的注释：
> Load and instantiate the factory implementations of the given type from "META-INF/spring.factories", using the configured class loader, the given argument resolver, and custom failure handling provided by the given failure handler.  
> The returned factories are sorted through AnnotationAwareOrderComparator.As of Spring Framework 5.3, if duplicate implementation class names are discovered for a given factory type, only one instance of the duplicated implementation type will be instantiated.  
> For any factory implementation class that cannot be loaded or error that occurs while instantiating it, the given failure handler is called.

```java
/**
 * Create a {@link SpringFactoriesLoader} instance that will load and
 * instantiate the factory implementations from the given location,
 * using the given class loader.
 * @param resourceLocation the resource location to look for factories
 * @param classLoader the ClassLoader to use for loading resources;
 * can be {@code null} to use the default
 * @return a {@link SpringFactoriesLoader} instance
 * @since 6.0
 * @see #forResourceLocation(String)
 */
public static SpringFactoriesLoader forResourceLocation(String resourceLocation, @Nullable ClassLoader classLoader) {
	Assert.hasText(resourceLocation, "'resourceLocation' must not be empty");
	ClassLoader resourceClassLoader = (classLoader != null ? classLoader :
			SpringFactoriesLoader.class.getClassLoader());
	Map<String, SpringFactoriesLoader> loaders = cache.computeIfAbsent(
			resourceClassLoader, key -> new ConcurrentReferenceHashMap<>());
	return loaders.computeIfAbsent(resourceLocation, key ->
			new SpringFactoriesLoader(classLoader, loadFactoriesResource(resourceClassLoader, resourceLocation)));
}

protected static Map<String, List<String>> loadFactoriesResource(ClassLoader classLoader, String resourceLocation) {
	Map<String, List<String>> result = new LinkedHashMap<>();
	try {
		Enumeration<URL> urls = classLoader.getResources(resourceLocation);
		while (urls.hasMoreElements()) {
			UrlResource resource = new UrlResource(urls.nextElement());
			Properties properties = PropertiesLoaderUtils.loadProperties(resource);
			properties.forEach((name, value) -> {
				String[] factoryImplementationNames = StringUtils.commaDelimitedListToStringArray((String) value);
				List<String> implementations = result.computeIfAbsent(((String) name).trim(),
						key -> new ArrayList<>(factoryImplementationNames.length));
				Arrays.stream(factoryImplementationNames).map(String::trim).forEach(implementations::add);
			});
		}
		result.replaceAll(SpringFactoriesLoader::toDistinctUnmodifiableList);
	}
	catch (IOException ex) {
		throw new IllegalArgumentException("Unable to load factories from location [" + resourceLocation + "]", ex);
	}
	return Collections.unmodifiableMap(result);
}

SpringFactoriesLoader 加载接口实现类的方法：
public <T> List<T> load(Class<T> factoryType, @Nullable ArgumentResolver argumentResolver,
		@Nullable FailureHandler failureHandler) {
	Assert.notNull(factoryType, "'factoryType' must not be null");
	List<String> implementationNames = loadFactoryNames(factoryType);
	logger.trace(LogMessage.format("Loaded [%s] names: %s", factoryType.getName(), implementationNames));
	List<T> result = new ArrayList<>(implementationNames.size());
	FailureHandler failureHandlerToUse = (failureHandler != null) ? failureHandler : THROWING_FAILURE_HANDLER;
	for (String implementationName : implementationNames) {
		T factory = instantiateFactory(implementationName, factoryType, argumentResolver, failureHandlerToUse);
		if (factory != null) {
			result.add(factory);
		}
	}
	AnnotationAwareOrderComparator.sort(result);
	return result;
}
```

### 通过 SpringApplication 的 setXXX() 方法自定义 SpringApplication
1. `setBeanNameGenerator(BeanNameGenerator beanNameGenerator)` ：设置自定义的 beanName 生成器
2. `setBannerMode(Banner.Mode bannerMode)`：设置启动 Banner 的模式：OFF、CONSOLE、LOG
3. `setBanner(Banner banner)`: 设置自定义 banner

## 扩展点

### SpringApplicationRunListener
`SpringApplication run()` 方法的侦听器。 `SpringApplicationRunListeners` 是通过 `SpringFactoriesLoader` 加载的，应该声明一个接受 `SpringApplication` 实例和`String[]``参数的公共构造函数。SpringApplication` 每次运行都会创建一个新的 `SpringApplicationRunListener` 实例。

`SpringApplicationRunListener` 的方法会在 `SpringApplication run()` 方法的不同阶段被调用。

```
public interface SpringApplicationRunListener {
	/**
	 * Called immediately when the run method has first started. Can be used for very
	 * early initialization.
	default void starting(ConfigurableBootstrapContext bootstrapContext) {
	}

	/**
	 * Called once the environment has been prepared, but before the
	default void environmentPrepared(ConfigurableBootstrapContext bootstrapContext,
			ConfigurableEnvironment environment) {
	}

	/**
	 * Called once the {@link ApplicationContext} has been created and prepared, but
	 * before sources have been loaded.
	default void contextPrepared(ConfigurableApplicationContext context) {
	}

	/**
	 * Called once the application context has been loaded but before it has been
	 * refreshed.
	default void contextLoaded(ConfigurableApplicationContext context) {
	}

	/**
	 * The context has been refreshed and the application has started but
	 * {@link CommandLineRunner CommandLineRunners} and {@link ApplicationRunner
	 * ApplicationRunners} have not been called.
	default void started(ConfigurableApplicationContext context, Duration timeTaken) {
		started(context);
	}

	/**
	 * Called immediately before the run method finishes, when the application context has
	 * been refreshed and all {@link CommandLineRunner CommandLineRunners} and
	 * {@link ApplicationRunner ApplicationRunners} have been called.
	default void ready(ConfigurableApplicationContext context, Duration timeTaken) {
		running(context);
	}

	/**
	 * Called immediately before the run method finishes, when the application context has
	 * been refreshed and all {@link CommandLineRunner CommandLineRunners} and
	 * {@link ApplicationRunner ApplicationRunners} have been called.
	 * @param context the application context.
	 * @since 2.0.0
	 * @deprecated since 2.6.0 for removal in 3.0.0 in favor of
	 * {@link #ready(ConfigurableApplicationContext, Duration)}
	 */
	@Deprecated
	default void running(ConfigurableApplicationContext context) {
	}

	/**
	 * Called when a failure occurs when running the application.
	 * @param context the application context or {@code null} if a failure occurred before
	 * the context was created
	default void failed(ConfigurableApplicationContext context, Throwable exception) {
	}

}
```

### ApplicationContextInitializer
回调接口，**用于在刷新 Spring ConfigurableApplicationContext 之前对其进行初始化**。
通常用于需要对应用上下文进行编程初始化的网络应用程序中。例如，针对上下文环境注册属性源或激活配置文件。请参阅 `ContextLoader` 和 `FrameworkServlet` 支持，分别用于声明 "contextInitializerClasses "上下文参数和初始参数。
我们鼓励 `ApplicationContextInitializer` 处理程序检测 Spring 的 Ordered 接口是否已实现或 @Order 注解是否存在，并在调用前对实例进行相应排序。

`ApplicationContextInitializer` 是在springboot启动过程(refresh方法前)调用,主要是在 `ApplicationContextInitializer` 中 `initialize` 方法中拉起了 `ConfigurationClassPostProcessor` 这个类(我在springboot启动流程中有描述)，通过这个 processor 实现了 beandefinition 。言归正传， `ApplicationContextInitializer` 实现主要有3种方式：

1. **使用spring.factories方式**

	首先我们自定义个类实现了 `ApplicationContextInitializer` ,然后在resource下面新建 `META-INF/spring.factories` 文件。然后在文件中加入：
	`org.springframework.context.ApplicationContextInitializer=com.baicy.springbootexplore.initializer.KeyInitializer`

2. **application.properties添加配置方式**

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

3. **直接通过 `SpringApplication` 的 `addXXX` 方法**

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
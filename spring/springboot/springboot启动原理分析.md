## Springboot 启动过程
{docsify-updated}

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
这些对象往往实现了许多自动配置的功能，此处涉及到Springboot 的 SPI 机制：SpringFactoriesLoader ：
private <T> List<T> getSpringFactoriesInstances(Class<T> type, ArgumentResolver argumentResolver) {
	return SpringFactoriesLoader.forDefaultResourceLocation(getClassLoader()).load(type, argumentResolver);
}

public static SpringFactoriesLoader forDefaultResourceLocation(@Nullable ClassLoader classLoader) {
	return forResourceLocation(FACTORIES_RESOURCE_LOCATION, classLoader);  # public static final String   FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
}

概括起来说就是 SpringApplication 构造时，会通过 SPI 机制，加载一些类，并设置到自身的相应的属性中。实例化好之后，就会调用 SpringApplication 对象的 run 方法：
public ConfigurableApplicationContext run(String... args) {
	long startTime = System.nanoTime();
	DefaultBootstrapContext bootstrapContext = createBootstrapContext();
	ConfigurableApplicationContext context = null;
	configureHeadlessProperty();
	SpringApplicationRunListeners listeners = getRunListeners(args); //获取SpringApplicationRunListener，然后在springboot启动过程中的不同阶段通知（调用）这些listener
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

#### SpringFactoriesLoader 
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
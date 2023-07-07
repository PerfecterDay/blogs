## Spring IOC
 {docsify-updated}
> https://docs.spring.io/spring-framework/reference/core/beans.html

- [Spring IOC](#spring-ioc)
	- [第一阶段的扩展点 BeanFactoryPostProcessor](#第一阶段的扩展点-beanfactorypostprocessor)
	- [第二阶段扩展 BeanPostProcessor](#第二阶段扩展-beanpostprocessor)
		- [Bean的生命周期](#bean的生命周期)
			- [BeanFactory中bean的生命周期](#beanfactory中bean的生命周期)
			- [ApplicationContext中bean的生命周期](#applicationcontext中bean的生命周期)
	- [Bean 的 scope](#bean-的-scope)
	- [Bean 注入的方式](#bean-注入的方式)
		- [Spring 循环依赖](#spring-循环依赖)
	- [classpath和classpath\*的区别](#classpath和classpath的区别)

Spring的IoC容器所起的作用就是它会以某种方式加载Configuration Metadata（通常也就是XML格式的配置信息），然后根据这些信息绑定整个系统的对象，最终组装成一个可用的基于轻量级容器的应用系统。
Spring的IoC容器实现以上功能的过程，基本上可以按照类似的流程划分为两个阶段，即**容器启动阶段**和**Bean实例化阶段**。
<center><img src="pics/ioc-phase.png" height=40% width=40%></center>

Spring的IoC容器在实现的时候，充分运用了这两个实现阶段的不同特点，在每个阶段都加入了相应的容器扩展点，以便我们可以根据具体场景的需要加入自定义的扩展逻辑。

1. 容器启动阶段  
    容器启动伊始，首先会通过某种途径加载 Configuration MetaData 。除了代码方式比较直接，在大部分情况下，容器需要依赖某些工具类（ BeanDefinitionReader ）对加载的 Configuration MetaData 进行解析和分析，并将分析后的信息编组为相应的 BeanDefinition，最后把这些保存了bean定义必要信息的BeanDefinition，注册到相应的BeanDefinitionRegistry，这样容器启动工作就完成了。总地来说，该阶段所做的工作可以认为是准备性的，重点更加侧重于对象管理信息的收集。当然，一些验证性或者辅助性的工作也可以在这个阶段完成。

2. Bean实例化阶段  
    容器会首先检查所请求的对象之前是否已经存在。如果没有，则会根据注册的BeanDefinition所提供的信息实例化被请求对象，并为其注入依赖。如果该对象实现了某些回调接口，也会根据回调接口的要求来装配它。当该对象装配完毕之后，容器会立即将其返回请求方使用。如果说第一阶段只是根据图纸装配生产线的话，那么第二阶段就是使用装配好的生产线来生产具体的产品了。

### 第一阶段的扩展点 BeanFactoryPostProcessor
Spring提供了一种叫做 BeanFactoryPostProcessor 的容器扩展机制。该机制允许我们在容器实例化相应对象之前，对注册到容器的 BeanDefinition 所保存的信息做相应的修改。这就相当于在容器实现的第一阶段最后加入一道工序，让我们对最终的 BeanDefinition 做一些额外的操作，比如修改其中bean定义的某些属性，为bean定义增加其他信息等。

`org.springframework.beans.factory.config.PropertyPlaceholderConfigurer` 和 `org.springframework.beans.factory.config.PropertyOverrideConfigurer` 是两个比较常用的 BeanFactoryPostProcessor。另外，为了处理配置文件中的数据类型与真正的业务对象所定义的数据类型转换， Spring还允许我们通过org.springframework.beans.factory.config.CustomEditorConfigurer 来注册自定义的 PropertyEditor 以补充容器中默认的 PropertyEditor 。

### 第二阶段扩展 BeanPostProcessor
在已经可以借助于 BeanFactoryPostProcessor 来干预 Spring 的第一个阶段启动之后，我们就可以开始探索下一个阶段，即bean实例化阶段的实现逻辑了。

#### Bean的生命周期
容器启动之后，并不会马上就实例化相应的bean定义。我们知道，容器现在仅仅拥有所有对象的 BeanDefinition 来保存实例化阶段将要用的必要信息。只有当请求方通过 BeanFactory 的 getBean() 方法来请求某个对象实例的时候，才有可能触发Bean实例化阶段的活动。 BeanFactory 的 getBean()法可以被客户端对象显式调用，也可以在容器内部隐式地被调用。隐式调用有如下两种情况:
+ 对于BeanFactory来说，对象实例化默认采用延迟初始化。通常情况下，当对象A被请求而需要第一次实例化的时候，如果它所依赖的对象B之前同样没有被实例化，那么容器会先实例化对象A所依赖的对象。这时容器内部就会首先实例化对象B，以及对象 A依赖的其他还没有实例化的对象。这种情况是容器内部调用getBean()，对于本次请求的请求方是隐式的。
+ ApplicationContext启动之后会实例化所有的bean定义，这个特性在本书中已经多次提到。但ApplicationContext在实现的过程中依然遵循Spring容器实现流程的两个阶段，只不过它会在启动阶段的活动完成之后，紧接着调用注册到该容器的所有bean定义的实例化方法getBean()。这就是为什么当你得到ApplicationContext类型的容器引用时，容器内所有对象已经被全部实例化完成。不信你查一下类org.AbstractApplicationContext的refresh()方法。

##### BeanFactory中bean的生命周期

<center><img src="pics/beanfactory-bean-lifecycle.png" height=40% width=40%></center>

BeanFactory 实例化一个 `bean` 的步骤：
<center><img src="pics/spring-bean.jpg" height=40% width=40%></center>

注意到：自定义的 `bean` 实例化时，会调用 `BeanPostProcessor`相关的方法，`BeanPostProcessor` 不一定是 `bean` 本身实现的接口，它是 Spring 容器提供的容器级别接口，所有实现 `BeanPostProcessor` 的类在注册到容器中后，都会在 `spring` 实例化某个`bean`的时候其作用。就是说，如果我们自定义了一个 `BeanPostProcessor` 的实现类并注册到容器中，则它会在`spring`实例化所有其它`bean`的时候起作用。

##### ApplicationContext中bean的生命周期
<center><img src="pics/applicationContext-bean-lifecycle.png" height=40% width=40%></center>

Bean 在应用上下文中的生命周期与在 BeanFactory 中的类似，实际上，应用上下文在初始化时，会向容器中注册一个 `ApplicationContextAwareProcessor` 类型的 `BeanPostProcessor` 实现类，并实现 `postProcessBeforeInitialization` 方法，这样的话，调用 `getBean`方法时， `ApplicationContextAwareProcessor` 就会生效， `postProcessBeforeInitialization` 判断如果 Bean 实现了 `EnvironmentAware` , `EmbeddedValueResolverAware` , `ResourceLoaderAware` , `ApplicationEventPublisherAware` , `MessageSourceAware` , `ApplicationContextAware` 接口，则会分别调用它们。

ApplicationContext 和 BeanFactory 的一个重大区别在于：前者会利用java反射机制自动识别出注册的 `BeanPostProcessor`, `BeanFactoryPostProcessor` , `InstantiationAwareBeanPostProcessor` ,并自动将它们注册到容器中；而后者需要手工调用 `addBeanPostProcessor` 方法注册它们。

`ApplicationContext` 的 `refresh` 方法，会在 `ApplicationContext` 初始化时调用：
```
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// 获取beanFactory，此时会解析加载xml中的beanDefinition，但是并没有注册 bean 对象
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

            // 注册一些 BeanPostProcessor bean，开始生成 bean
			prepareBeanFactory(beanFactory);

            // Allows post-processing of the bean factory in context subclasses.
            postProcessBeanFactory(beanFactory);

            // 注册并调用BeanFactoryPostProcessor的postProcessBeanFactory方法
            invokeBeanFactoryPostProcessors(beanFactory);

            // 注册 BeanPostProcessor 类型的bean，自动实现 BeanPostProcessor 注册
            registerBeanPostProcessors(beanFactory);

            // Initialize message source for this context.
            initMessageSource();

            // Initialize event multicaster for this context.
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            onRefresh();

            // Check for listener beans and register them.
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            finishRefresh();
    }
```
看看几个主要的方法：

1. `obtainFreshBeanFactory()` 方法 :

	```
	protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
		refreshBeanFactory();
		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
		if (logger.isDebugEnabled()) {
			logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);
		}
		return beanFactory;
	}

	@Override
	protected final void refreshBeanFactory() throws BeansException {
		if (hasBeanFactory()) {
			destroyBeans();
			closeBeanFactory();
		}
		DefaultListableBeanFactory beanFactory = createBeanFactory();
		beanFactory.setSerializationId(getId());
		customizeBeanFactory(beanFactory);
		//加载beanDefinition
		loadBeanDefinitions(beanFactory);
		synchronized (this.beanFactoryMonitor) {
			this.beanFactory = beanFactory;
		}
	}
	```
	该方法主要是实例化一个 `beanFactory`，并加载 bean 定义。

2. `prepareBeanFactory`:
```
    protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		// Tell the internal bean factory to use the context's class loader etc.
		beanFactory.setBeanClassLoader(getClassLoader());
		beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
		beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

		// Configure the bean factory with context callbacks.
		beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
		beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
		beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
		beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
		beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

		// BeanFactory interface not registered as resolvable type in a plain factory.
		// MessageSource registered (and found for autowiring) as a bean.
		beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
		beanFactory.registerResolvableDependency(ResourceLoader.class, this);
		beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
		beanFactory.registerResolvableDependency(ApplicationContext.class, this);

		// Register early post-processor for detecting inner beans as ApplicationListeners.
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

		// Detect a LoadTimeWeaver and prepare for weaving, if found.
		if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			// Set a temporary ClassLoader for type matching.
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}

		// Register default environment beans.
		if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
		}
	}
```
3. `registerListeners`:
```
    protected void registerListeners() {
		// Register statically specified listeners first.
		for (ApplicationListener<?> listener : getApplicationListeners()) {
			getApplicationEventMulticaster().addApplicationListener(listener);
		}

		// Do not initialize FactoryBeans here: We need to leave all regular beans
		// uninitialized to let post-processors apply to them!
		String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
		for (String listenerBeanName : listenerBeanNames) {
			getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
		}

		// Publish early application events now that we finally have a multicaster...
		Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
		this.earlyApplicationEvents = null;
		if (earlyEventsToProcess != null) {
			for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
				getApplicationEventMulticaster().multicastEvent(earlyEvent);
			}
		}
	}
```
### Bean 的 scope
Spring容器最初提供了两种bean的scope类型：`singleton` 和 `prototype` ，但发布2.0之后，又引入了另外三种scope类型，即 `request` 、 `session` 和 `global session` 类型。不过这三种类型有所限制，只能在Web应用中使用。
+ `singleton`: 标记为拥有singleton scope的对象定义，在Spring的IoC容器中只存在一个实例，所有对该对象的引用将共享这个实例。该实例从容器启动，并因为第一次被请求而初始化之后，将一直存活到容器退出，也就是说，它与IoC容器“几乎”拥有相同的“寿命”。
+ `prototype` ：容器在接到该类型对象的请求的时候，会每次都重新生成一个新的对象实例给请求方。虽然这种类型的对象的实例化以及属性设置等工作都是由容器负责的，但是只要准备完毕，并且对象实例返回给请求方之后，容器就不再拥有当前返回对象的引用，请求方需要自己负责当前返回对象的后继生命周期的管理工作，包括该对象的销毁。也就是说，容器每次返回给请求方一个新的对象实例之后，就任由这个对象实例“自生自灭”了。
+ `request`: Spring容器的 XmlWebApplicationContext 会为每个HTTP请求创建一个全新的bean对象供当前请求使用，当请求结束后，该对象实例的生命周期即告结束。
+ `session` ：Spring容器会为每个独立的session创建属于它们自己的全新的bean对象实例。与request相比，除了可能更长的存活时间，其他方面真是没什么差别。
+ `global session`: 只有应用在基于portlet的Web应用程序中才有意义，它映射到portlet的global范围的 session。如果在普通的基于servlet的Web应用中使用了这个类型的scope，容器会将其作为普通的session类型的scope对待。

### Bean 注入的方式
三种依赖注入的方式，即构造方法注入（constructor injection）、setter方法注入（setter injection）以及接口注入（interface injection）。

1. 构造方法注入  
   构造方法注入，就是被注入对象可以通过在其构造方法中声明依赖对象的参数列表，让外部（通常是IoC容器）知道它需要哪些依赖对象。IoC Service Provider会检查被注入对象的构造方法，取得它所需要的依赖对象列表，进而为其注入相应的对象。同一个对象是不可能被构造两次的，因此，被注入对象的构造乃至其整个生命周期，应该是由IoC Service Provider来管理的。构造方法注入方式比较直观，对象被构造完成后，即进入就绪状态，可以马上使用。
2. setter 方法注入  
   对于JavaBean对象来说，通常会通过setXXX()和getXXX()方法来访问对应属性。setter 方法注入就是通过 setter 方法将相应的依赖对象设置到被注入对象中。setter方法注入虽不像构造方法注入那样，让对象构造完成后即可使用，但相对来说更宽松一些，可以在对象构造完成后再注入。
3. 接口注入  
   相对于前两种注入方式来说，接口注入没有那么简单明了。被注入对象如果想要IoC Service Provider为其注入依赖对象，就必须实现某个接口。这个接口提供一个方法，用来为其注入依赖对象。IoC Service Provider最终通过这些接口来了解应该为被注入对象注入什么依赖对象。

#### Spring 循环依赖
首先，Spring内部维护了三个Map，也就是我们通常说的三级缓存。
在Spring的`DefaultSingletonBeanRegistry`类中，你会赫然发现类上方挂着这三个Map：
+ `singletonObjects`: 它是我们最熟悉的朋友，俗称“单例池”“容器”，缓存创建完成单例Bean的地方。
+ `singletonFactories`: 映射创建Bean的原始工厂
+ `earlySingletonObjects`: 映射Bean的早期引用，也就是说在这个Map里的Bean不是完整的，甚至还不能称之为 “Bean”，只是一个Instance。

后两个Map其实是“垫脚石”级别的，只是创建Bean的时候，用来借助了一下，创建完成就清掉了。
为什么成为后两个Map为垫脚石，假设最终放在singletonObjects的Bean是你想要的一杯“凉白开”。
那么Spring准备了两个杯子，即singletonFactories和earlySingletonObjects来回“倒腾”几番，把热水晾成“凉白开”放到singletonObjects中。

简言之，两个池子：一个成品池子 singletonObjects ，一个半成品池子 earlySingletonObjects 。能解决循环依赖的前提是：spring开启了allowCircularReferences，那么一个正在被创建的bean才会被放在半成品池子里。在注入bean，向容器获取bean的时候，优先向成品池子要，要不到，再去向半成品池子要。

但是对于构造器注入来说，因为必须调用构造器来实例化对象，但是构造器有循环依赖，所以没有办法构造实例化对象，也就没法讲实例对象放到半成品池子，所以 spring 无法解决构造器的循环依赖问题。

### classpath和classpath*的区别

记录一下踩坑：
有两个项目A和B，A依赖B，最终运行的是A。使用mybatis做数据库访问，在一个B的jar包中写好了mapper.xml文件，A项目运行时，能运行A中的mapper文件，找不到B中绑定的sql语句。就是说mybatis扫描mapper.xml文件时，能扫描到项目本身的mapper，却不能扫描到依赖jar包中的mapper，仔细查看文件才发现配置文件有问题：

    mapper-locations: classpath:mapper/**/*.xml
改为

    mapper-locations: classpath*:mapper/**/*.xml
后，问题解决。

classpath和classpath*区别： 
1. classpath：只会到你的class路径中查找找文件。
2. classpath*：不仅包含class路径，还包括jar文件中（class路径）进行查找。

注意： 用classpath*:需要遍历所有的classpath，所以加载速度是很慢的；因此，在规划的时候，应该尽可能规划好资源文件所在的路径，尽量避免使用classpath*。
# Springboot 中的 ApplicationContextInitializer
{docsify-updated}


`ApplicationContextInitializer` 是在springboot启动过程(refresh方法前)调用,主要是在 `ApplicationContextInitializer` 中 `initialize` 方法中拉起了 `ConfigurationClassPostProcessor` 这个类(我在springboot启动流程中有描述)，通过这个 processor 实现了 beandefinition 。言归正传， `ApplicationContextInitializer` 实现主要有3中方式：

### 使用spring.factories方式
首先我们自定义个类实现了 `ApplicationContextInitializer` ,然后在resource下面新建 `META-INF/spring.factories` 文件。然后在文件中加入：
`org.springframework.context.ApplicationContextInitializer=com.baicy.springbootexplore.initializer.KeyInitializer`

### application.properties添加配置方式
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

### 直接通过 `SpringApplication` 的 `add` 方法
```
public static void main(String[] args) {
    SpringApplication application = new SpringApplication(SpringbootExploreApplication.class);
    application.addInitializers(new KeyInitializer());
    application.run(args);
}
```
--------------------- 
原文：https://blog.csdn.net/leileibest_437147623/article/details/81074174 

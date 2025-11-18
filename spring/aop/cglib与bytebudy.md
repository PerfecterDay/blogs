# Spring AOP 核心实现原理
{docsify-updated}

## cglib
Spring 的 CGLIB 是一个“Shaded版本”， 你会在 Spring 里看到：
```
org.springframework.cglib.proxy.Enhancer
org.springframework.cglib.core.ReflectUtils
...
```
这是从 CGLIB 源码 整体拷贝过来并重打包（shade）为 Spring 自己的包名，避免与外部 CGLIB 冲突。

原本 CGLIB 是一个独立库：`net.sf.cglib.proxy.Enhancer` ,Spring 早期直接依赖它。但 CGLIB 不维护了，而 Spring 仍需要它，而且常常被用来做类代理。所以 Spring 团队做了几步：
1.	把 CGLIB 源码复制一份（完全合法，因为 CGLIB 使用宽松的 Apache License 2.0）
2.	修复与 Java 新版本不兼容的问题，比如模块系统、反射、MethodHandles 等
3.	并把包名改成 org.springframework.cglib 避免冲突和二义性

```
public static void main(String[] args) {
    Enhancer enhancer = new Enhancer();
    enhancer.setSuperclass(TargetClass.class);
    enhancer.setCallback(new MethodInterceptor() {
        @Override
        public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
            System.out.println("before");
            Object result = proxy.invokeSuper(obj, args);
            System.out.println("after");
            return result;
        }
    });
    TargetClass proxy = (TargetClass) enhancer.create();
}
```

核心接口： `MethodInterceptor` 是 CGLIB 的“切面”入口：
```
public interface MethodInterceptor {
    Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy)
}
```

## Spring 对 CGLIB 的封装与增强
Spring 并不会直接让你调用 Enhancer，而是封装在 AOP 框架中：
+ `ProxyFactory`
+ `CglibAopProxy`
+ `DefaultAopProxyFactory`

```
ProxyFactory proxyFactory = new ProxyFactory(targetObj);
//或者 proxyFactory.setTarget(targetObj);
proxyFactory.addAdvisor(advisor);
//或者 proxyFactory.addAdvice(advice);
Object proxy = proxyFactory.getProxy();
```

## bytebudy
ByteBuddy 是 CGLIB 的后继者。Spring 已经在逐步替换 CGLIB，但是：
+ Spring 仍可能在某些 class 级代理上回退到 CGLIB（主要是为了兼容旧版本）
+ Spring Cloud（比如 RefreshScope）仍使用 CGLIB 代理
+ 所以 Spring 内部依然保留了这个 CGLIB 的 shaded 版本

这就是 `org.springframework.cglib.proxy.Enhancer` 存在的原因。


## AOP 核心实现原理
CglibAopProxy 在 Spring 启动过程中的工作机制

本文详细介绍 CglibAopProxy 在 Spring 启动过程中是如何起作用的，重点解释其在自动代理（Auto-Proxy）机制中的角色和工作流。

⸻

🧠 总体流程概览

Spring Boot 启动期间，Bean 的代理创建流程一般会经历以下阶段：
1. 扫描 Bean Definitions
2. 实例化 Bean
3. 执行 BeanPostProcessor（重要！）
4. 为符合条件的 Bean 创建代理对象（CglibAopProxy 或 JDK 动态代理）
5. 将代理对象注册到容器中供业务使用

⸻

🔍 核心触发点： `BeanPostProcessor`

1. 注册 AOP 自动代理器
当使用 AOP（如 @EnableAspectJAutoProxy，或通过 Spring Boot 引入 AOP 启动器）时，Spring 注册 `AnnotationAwareAspectJAutoProxyCreator` ：

`AnnotationAwareAspectJAutoProxyCreator`

它实现了 BeanPostProcessor，会在每个 Bean 初始化后进行增强判断并创建代理。

⸻

2. 判断是否需要代理：wrapIfNecessary()

Spring 在 Bean 初始化后的 postProcessAfterInitialization 中调用：

wrapIfNecessary(bean, beanName)

该方法会：
+ 检查 Bean 是否匹配 AOP 切点（如 @Transactional, @Before, @Around）
+ 如果匹配，则生成代理对象

⸻

3. 使用 ProxyFactory 创建代理对象

代理创建逻辑通过 ProxyFactory 完成：

ProxyFactory proxyFactory = new ProxyFactory(bean);
proxyFactory.addAdvisors(advisors);
proxyFactory.setProxyTargetClass(true); // 或根据条件处理

然后调用：

return proxyFactory.getProxy();


⸻

🔧 代理策略：选择 CglibAopProxy

ProxyFactory.getProxy() 会委托给：

AopProxy aopProxy = createAopProxy();

由 DefaultAopProxyFactory 决定代理策略：
	•	若目标类没有实现接口，或使用 proxyTargetClass = true 强制使用 CGLIB，则选择：

new CglibAopProxy(config)

否则采用 JDK 动态代理。

⸻

🧩 CglibAopProxy 如何生成代理？

方法：getProxy()

通过 CGLIB 的 Enhancer 动态生成代理子类：

Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(targetClass);        // 继承目标类
enhancer.setCallbackTypes(callbacks);       // 设置拦截器
enhancer.setCallbackFilter(callbackFilter); // 拦截器选择策略
return enhancer.create();                   // 动态生成代理

方法调用拦截：MethodInterceptor

生成的子类会重写所有非-final 的方法，并在方法调用时通过 MethodInterceptor（如 DynamicAdvisedInterceptor）执行增强逻辑：

methodProxy.invokeSuper(proxy, args);

执行顺序通常为：
	•	执行切面前置逻辑（如 @Before）
	•	调用原目标方法
	•	执行切面后置逻辑

⸻

📊 整体流程图

Spring Boot 启动
        ↓
加载 Bean Definition
        ↓
注册 AOP 自动代理器（AnnotationAwareAspectJAutoProxyCreator）
        ↓
Bean 创建后 → wrapIfNecessary()
        ↓
匹配切点 → 构造 ProxyFactory
        ↓
使用 CglibAopProxy 创建代理子类
        ↓
代理对象替换原 Bean 注入容器
        ↓
业务逻辑调用时 → 通过代理拦截执行切面


⸻

🌀 常见应用场景
	•	@Transactional 注解的方法和类
	•	@Aspect 定义的切面增强逻辑
	•	@Configuration 配置类中的内部方法调用
	•	手动配置了 proxyTargetClass = true 强制使用 CGLIB

⸻

📎 总结

CglibAopProxy 在 Spring 启动过程中主要负责：
	•	在 Bean 创建后，判断是否需要对 Bean 进行增强
	•	使用 CGLIB 动态生成代理子类，对方法调用进行拦截
	•	实现 AOP 切面逻辑，无需修改原始业务代码

理解这一机制对调试 Spring AOP 问题、性能优化、理解代理生成逻辑具有重要意义。

如果你需要进一步深入查看代理类字节码或对比 JDK 动态代理的行为，也可以继续探索和调试。
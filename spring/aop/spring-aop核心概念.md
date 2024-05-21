# AOP 核心概念
{docsify-updated}

## 连接点(join point)
程序运行中的一些时间点, 例如一个方法的执行，构造方法的调用，属性设置及获取，或者是一个异常的处理等.但是在 Spring AOP 中, `join point` 总是方法的执行点, 即只支持方法连接点,不可以在类上增强。

## 切点(point cut)
匹配 `join point` 的谓词(a predicate that matches join points).
`Advice` 是和特定的 `point cut` 关联的, 并且在 `point cut` 相匹配的 `join point` 中执行. 
在 Spring 中, 所有的方法都可以认为是 `joinpoint`, 但是我们并不希望在所有的方法上都添加 `Advice`, 而 `pointcut` 的作用就是提供一组规则(使用 AspectJ pointcut expression language 来描述) 来匹配`joinpoint`, 给满足规则的 `joinpoint` 添加 `Advice`.

简而言之，切点就是用来筛选哪些连接点需要织入增强代码的。  
Pointcut接口的声明如下：
```java
public interface  {
    Pointcut TRUE = TruePointcut.INSTANCE;

    ClassFilter getClassFilter();

    MethodMatcher getMethodMatcher();
}
```
ClassFilter 和 MethodMatcher 分别用于匹配将被织入横切逻辑的对象和相应的方法，即类型匹配和方法匹配。
<center><img src="/pics/Pointcut.png" width=50% alt=""></center>

常见的 Pointcut:
1. `NameMatchMethodPointcut`
2. `JdkRegexpMethodPointcut`
3. `AnnotationMatchingPointcut`
4. `ComposablePointcut`

## 关于 join point 和 point cut 的区别
在 Spring AOP 中, 所有的方法执行都是 `join point`. 而 `point cut` 是一个描述信息, 它修饰的是 `join point`, 通过 `point cut`, 我们就可以确定哪些 `join point` 可以被织入 Advice. 因此 `join point` 相当于数据库中的记录， 而 `point cut` 相当于带有查询条件查询出来的记录。
`advice` 是在 `join point `上执行的, 而 `point cut` 规定了哪些 `join point` 可以执行那些 `advice`.

## 增强-Advice
按照`Advice`实例能否在目标对象类的所有实例间共享这一标准，可以将`Advice`分为 `per-class` 和 `per-instance `类型：
+ per-class 类型的 Advice，可以在所有目标类对象之间共享，不会保存任何目标对象的状态或者添加新特性。

  1. `Before Advice`: 在连接点指定位置之前执行的增强类型。

  2. `After Advice`: 在连接点指定位置之后执行的增强类型，该类型还有三种子类型：
     1. After returning advice: 只有切点处执行流程正常完成后，才会执行
     2. After throwing advice: 只有切点处执行流程抛出异常的情况下，才会执行
     3. After advice: 不管切点是正常执行还是异常了都会执行，相当于 finally 块一样。
     
  3. `Around Advice（Interceptor）`: 可以在切点之前和之后都指定相应的逻辑，甚至于中断或者忽略切点原来的程序流程的执行。Spring中没有直接定义对应的 Around Advice 的实现接口，而是直接采用 AOP Alliance的标准接口 `org.aopalliance.intercept.MethodInterceptor` , 这个接口只有一个 `invoke` 方法：

    ```java
    public interface MethodInterceptor extends Interceptor{
    	Object invoke(MethodInvocation invocation) throws Throwable;
    }
    ```
<center><img src="/pics/Advice.png" width=60% alt=""></center>

+ `per-instance` 类型的 Advice，Advice对象不会在目标类对象的实例之间共享。不同的目标对象创建不同的`Advice`实例。在 Spring AOP 中， `Introduction` 是唯一一种 `per-instance` 类型的 Advice 。  

  `Introduction`: 可以在不改动目标类定义的情况下，为原有的对象添加新的特性或者行为。在Spring中，为目标对象添加新的属性和行为必须声明相应的接口以及相应的实现。这样，再通过特定的拦截器将新的接口定义及实现类中的逻辑附加到目标对象上，目标对象就拥有了新的状态和行为。Spring中这个拦截器就是 ` IntroductionInterceptor` 接口来实现。
  <center><img src="/pics/aop-introduction.png" width=60% alt=""></center>

## 切面 (aspect)
`aspect` 由 `pointcut` 和 `advice` 组成, 它既包含了横切逻辑(增强)的定义, 也包括了切点的定义. Spring AOP就是负责实施切面的框架, 它将切面所定义的横切逻辑织入到切面所指定的连接点中.
AOP的工作重心在于如何将增强织入目标对象的连接点上, 这里包含两个工作:

1. 如何通过 `pointcut` 和 `advice` 定位到特定的 joinpoint 上
2. 如何在 `advice` 中编写切面代码.

Spring 中使用 `org.springframework.aop.Advisor` 接口表示切面。可以划分为 `PointcutAdvisor `和 `IntroductionAdvisor` 两个分支。
1. PointcutAdvisor
	<center><img src="/pics/Advisor.png" width=60% alt=""></center>
2. `IntroductionAdvisor`  

    `IntroductionAdvisor` 的类层次比较简单，只有一个 `DefaultIntroductionAdvisor` 的默认实现。

## 织入(Weaving)
将 aspect 和其他对象连接起来, 并创建 adviced object 的过程.

根据不同的实现技术, AOP织入有三种方式:

* 编译器织入, 这要求有特殊的Java编译器.AspectJ的 ajc 编译器
* 类装载期织入, 这需要有特殊的类装载器. Jboss AOP 自定义的 ClassLoader
* 动态代理织入, 在运行期为目标类添加增强(Advice)生成代理类的方式.

Spring 采用动态代理织入, 而AspectJ采用编译器织入和类装载期织入。
上述所有的概念可以由下面这张图大致概括：
<center><img src="pics/spring-aop.png" alt="" height=60% width=60%></center>

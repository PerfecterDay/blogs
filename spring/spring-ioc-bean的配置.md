# Spring IoC Bean的配置
{docsify-updated}

## Bean 注入的方式
三种依赖注入的方式，即**构造方法注入**（constructor injection）、**setter方法注入**（setter injection）以及**接口注入**（interface injection）。

1. 构造方法注入  
   构造方法注入，就是被注入对象可以通过在其构造方法中声明依赖对象的参数列表，让外部（通常是IoC容器）知道它需要哪些依赖对象。IoC Service Provider会检查被注入对象的构造方法，取得它所需要的依赖对象列表，进而为其注入相应的对象。同一个对象是不可能被构造两次的，因此，被注入对象的构造乃至其整个生命周期，应该是由IoC Service Provider来管理的。构造方法注入方式比较直观，对象被构造完成后，即进入就绪状态，可以马上使用。
2. setter 方法注入  
   对于JavaBean对象来说，通常会通过setXXX()和getXXX()方法来访问对应属性。setter 方法注入就是通过 setter 方法将相应的依赖对象设置到被注入对象中。setter方法注入虽不像构造方法注入那样，让对象构造完成后即可使用，但相对来说更宽松一些，可以在对象构造完成后再注入。
3. 接口注入  
   相对于前两种注入方式来说，接口注入没有那么简单明了。被注入对象如果想要IoC Service Provider为其注入依赖对象，就必须实现某个接口。这个接口提供一个方法，用来为其注入依赖对象。IoC Service Provider最终通过这些接口来了解应该为被注入对象注入什么依赖对象。

### Spring 循环依赖
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

## Bean 的 scope
Spring容器最初提供了两种bean的scope类型：`singleton` 和 `prototype` ，但发布2.0之后，又引入了另外三种scope类型，即 `request` 、 `session` 和 `global session` 类型。不过这三种类型有所限制，只能在Web应用中使用。
+ `singleton`: 标记为拥有singleton scope的对象定义，在Spring的IoC容器中只存在一个实例，所有对该对象的引用将共享这个实例。该实例从容器启动，并因为第一次被请求而初始化之后，将一直存活到容器退出，也就是说，它与IoC容器“几乎”拥有相同的“寿命”。
+ `prototype` ：容器在接到该类型对象的请求的时候，会每次都重新生成一个新的对象实例给请求方。虽然这种类型的对象的实例化以及属性设置等工作都是由容器负责的，但是只要准备完毕，并且对象实例返回给请求方之后，容器就不再拥有当前返回对象的引用，请求方需要自己负责当前返回对象的后继生命周期的管理工作，包括该对象的销毁。也就是说，容器每次返回给请求方一个新的对象实例之后，就任由这个对象实例“自生自灭”了。
+ `request`: Spring容器的 XmlWebApplicationContext 会为每个HTTP请求创建一个全新的bean对象供当前请求使用，当请求结束后，该对象实例的生命周期即告结束。
+ `session` ：Spring容器会为每个独立的session创建属于它们自己的全新的bean对象实例。与request相比，除了可能更长的存活时间，其他方面真是没什么差别。
+ `global session`: 只有应用在基于portlet的Web应用程序中才有意义，它映射到portlet的global范围的 session。如果在普通的基于servlet的Web应用中使用了这个类型的scope，容器会将其作为普通的session类型的scope对待。


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
## multi-protocol-framework-springboot
SpringBootAutoConfiguration 中注入了以下 bean:
+ SgPlaceHolder
+ MultiSgSyncClientProcessor ： 处理 @MultiSgSyncClient 注解
+ SgSpringListener
+ SgSpringConfig ： 当使用 XML 配置时生效
+ ConfigurationContainer ： 使用 yaml 配置时生效
+ SgMultiLifecycle ：


### SgSpringListener
SgSpringListener 监听 ContextRefreshedEvent/ContextClosedEvent

### SgMultiLifecycle

SgMultiLifecycle.startup

### MultiSgSyncClientProcessor
MultiSgSyncClientProcessor 继承自 SmartInstantiationAwareBeanPostProcessor
    SynchronousConsumerManager.getConsumerInstance(injectedType, builder.build()); 生成bean
        Object object = Proxy.newProxyInstance(clazz.getClassLoader(),
                new Class<?>[]{clazz}, new SynchronousConsumerInvocationHandler<>(clazz, rpcOptions)); 使用JDK动态代理生成代理对象

                 ClientInvokeTargetFilter->SgMultiHttpConnection 发起实际调用
                 Object proxy = Forest.config(providerNode.getIdentifier()).createInstance(clazzName); 使用 Forest 发起远程调用
                 return methodName.invoke(proxy, args);



HardWareUtil 获取主机IP


配置类： `ConfigurationContainer`


注册参数：
```
app:unknown
metadata:{"side":"provider","appVersion":"1.0.0","pid":"98390","proxyHost":"","timeout":"5000","timeStamp":"1739861131989","retryTimes":"0","proxyPort":"","protocol":"http","application":"testChain","framework.version":"3.0.0-SNAPSHOT","internalVersion":"3.0.0.7","tag":"","lang":"java","dcCity":"SH","useTLS":"false","dc":"CS"}
ip:10.176.115.168
weight:100.0
ephemeral:true
serviceName:DEFAULT_GROUP@@testChain
accessToken:eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJuYWNvcyIsImV4cCI6MTczOTg3OTEzMn0.gfKa3hI0o1SSWuRxqY5kbz1H28AvCahTqFDVQMddUo8
groupName:DEFAULT_GROUP
namespaceId:http-application
port:8099
enable:true
healthy:true
clusterName:
```


## multi-protocol-framework-http-starter
`HttpProviderInvokeFilter` 拦截 http 的调用

## SPIContainer
`SPIContainer` 自定义的 SPI，会扫描 jar 包中的 `META-INF/gtja-multi-sg-services/` 路径下注册的  

## 探活
SgMultilHttpExtConnectionPool -> orgValidate

sg-monitor-api 注册了 META-INF/gtja-multi-sg-services/com.gtja.sg.multi.monitor.SgMonitorBehavior SPI, `basic=com.gtja.sg.multi.monitor.BasicSgMonitorBehavior`
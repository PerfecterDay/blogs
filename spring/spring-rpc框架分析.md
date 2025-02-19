RPCCtxRequest

SgSpringListener 监听 ContextRefreshedEvent/ContextClosedEvent

调用
SgMultiLifecycle.startup

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
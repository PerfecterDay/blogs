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
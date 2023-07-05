# Spring MVC 中 RequestMappingHandlerAdapter 分析
{docsify-updated}


## handleInternal 方法
在 handleInternal 方法中最主要的是调用 invokeHandlerMethod 方法。

## invokeHandlerMethod 方法
此方法是整个类的核心方法，大部分处理流程在这个方法中完成。

    WebDataBinderFactory binderFactory = getDataBinderFactory(handlerMethod);
    ModelFactory modelFactory = getModelFactory(handlerMethod, binderFactory);
    ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod);
    if (this.argumentResolvers != null) {
        invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
    }
    if (this.returnValueHandlers != null) {
        invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
    }
    invocableMethod.setDataBinderFactory(binderFactory);
    invocableMethod.setParameterNameDiscoverer(this.parameterNameDiscoverer);
    ModelAndViewContainer mavContainer = new ModelAndViewContainer();
    ......
    invocableMethod.invokeAndHandle(webRequest, mavContainer);
有几个重要的对象 `WebDataBinderFactory` , `ModelFactory` , `ModelAndViewContainer` ,`ServletInvocableHandlerMethod` 。


```
ServletInvocableHandlerMethod.invokeAndHandle -> InvocableHandlerMethod.invokeForRequest ->
    Object[] args = InvocableHandlerMethod.getMethodArgumentValues -> 				args[i] = HandlerMethodArgumentResolverComposite.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
    ->ModelAttributeMethodProcessor.resolveArgument()
        			WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);
                    bindRequestParameters(binder, webRequest);
				    validateIfApplicable(binder, parameter);-> validator.validate();
                    return args;

    return doInvoke(args);        
```
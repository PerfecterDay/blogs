## Spring IOC 注解
 {docsify-updated}
> https://docs.spring.io/spring-framework/reference/core/beans.html

- [Spring IOC 注解](#spring-ioc-注解)
	- [常用注解](#常用注解)


### 常用注解
`@Resource` ： 默认是根据bean名字注入的，
```
	//会注入名字为 tradeLoginService 的 bean，如果这个bean 不是 TradeLoginServiceV2 类型就会报错
    @Resource
    private TradeLoginServiceV2 tradeLoginService;

	//会注入名字为 tradeLoginServiceV2 的 bean
	@Resource(name = "tradeLoginServiceV2")
    private TradeLoginServiceV2 tradeLoginService;
```
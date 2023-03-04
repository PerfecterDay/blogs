## 使用Spring 的异步线程池 @Async
{docsify-updated}

- [使用Spring 的异步线程池 @Async](#使用spring-的异步线程池-async)
	- [使用Spring Async的步骤](#使用spring-async的步骤)
	- [@Async 和 @Cacheable 不能同时用于一个方法](#async-和-cacheable-不能同时用于一个方法)


### 使用Spring Async的步骤
1. 在一个配置类中加上 @EnableAsync 注解
	```
	@Configuration
	@PropertySource(value = "classpath:/configuration/${env}/pool.yml", factory = YamlPropertySourceFactory.class)
	@ConfigurationProperties(prefix = "task.pool")
	@Data
	@EnableAsync
	public class AsyncThreadPoolConfig implements AsyncConfigurer {
		private Integer coreSize;
		private Integer maxSize;
		private Integer queueSize;
		private Integer keepAlive;

		//自定义线程池
		@Bean
		public Executor asyncPool() {
			ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
			executor.setCorePoolSize(coreSize);
			executor.setMaxPoolSize(maxSize);
			executor.setQueueCapacity(queueSize);
			executor.setKeepAliveSeconds(keepAlive);
			return executor;
		}
	}
	```
2. 使用自定义线程池有两种方式，
   1. 局部注入：定义一个线程池的bean 后，在@Async注解后指定要使用的线程池的名字
   2. 全局注入：在注入线程池bean的配置类上实现`AsyncConfigurer`	接口，这样注入的线程池就是默认的线程池了（见上面的代码）
3. 在需要异步执行的方法上加上 @Async 注解，注意不能在定义这个异步方法类的内部其他方法中调用这个方法，否则会失效。（AOP增强失效的原因）
   ```
    @Async
    @Cacheable(value = "smsCode")
    public void saveSms(String mobile, String code) {
        log.info(Thread.currentThread().getName());
        SmsEntity sms = new SmsEntity(mobile,code,"login");
        save(sms);
    }
   ```

### @Async 和 @Cacheable 不能同时用于一个方法
As I said, the two annotations do not work together on the same method. If you think about it, it is obvious. Marking with @Async is also @Cacheable means to delegate the whole cache management to different asynchronous threads. If the computation of the value of the CompletableFuture will take a long time to complete, the value in the cache will be placed after that time by Spring Proxy.
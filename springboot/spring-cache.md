## Spring Cache的集成
{docsify-updated}

### 缓存基础知识
1. 缓存命中率
	即从缓存中 读取数据的次数与总读取 次数的 比率。一般来说，命中率越高越好。 
	1. 命中率 = 从缓存中读取的次数/总读取次数(从缓存中读取的次数+从慢速设备上读取的次数)
	2. Miss率 = 没有从缓存中读取的次数/总读取次数(从缓存中读取的次数+从慢速设备上读取的次数)

2. 缓存过期策略
	即如果缓存满了，从缓存中移除数据的策略。常见的有 LFU 、 LRU 、 FIFO 。
	+ FIFO(FirstInFirstOut):先进先出策略，即先放入缓存的数据先被移除。
	+ LRU(LcastRecentlyUsed):最久未使用策略，即使用时间距高现在最久的那个数据被移除。
	+ LFU(LeastFrequentlyUsed):最近最少使用策略，即一定时间段内使用次数(频率)最少的那个数据被移除。
	+ TMTL(TimeToLive):存活期，即从缓存中创建时间点开始直至到期的一个时间段(不管在这个时间段内有没有访问都将过期)。
	+ TTI(TimeToldle):空闲期，即一个数据多久没被访问就从缓存中移除的时间。

### 自己实现缓存的简单实现
```
public class CacheManager<T> {
    private Map<String, T> cache = new ConcurrentHashMap<String, T>();

    public T getValue(Object key) {
        return cache.get(key);
    }

    public void addOrUpdateCache(String key, T value) {
        cache.put(key, value);
    }

    public void evictcache(String key) {
        if (cache.containsKey(key)) {
            cache.remove(key);
        }
    }

    public void evictCache() {
        cache.clear();
    }
}

public class UserService {
    private CacheManager<User> cacheManager;

    public UserService() {
        //构道一个级存節理器
        cacheManager = new CacheManager<>();
    }
    public User getUserById(String userId) {
        //首先查询變存
        User result = cacheManager.getValue(userId);
        if (result != null) {
            System.out.println("get from cache... . " + userId);
            return result;
        }
        result = getFromDB(userId);
        if (result != null) {
            //将结果更新到缓存中去
            cacheManager.addOrUpdateCache(userId, result);
        }
        return result;
    }

    public void reload() {
        cacheManager.evictCache();
    }

    private User getFromDB(String userId) {
        System.out.println("real querying db..." + userId);
        return new User(userId);
    }
}
```

使用SpringCache带来的好处如下:
+ 支持开箱即用(Out-Of-The-Box)，并提供基本的Cache抽象，方便切换各种底层Cache。
+ 类似于Spring提供的数据库事务管理，通过Cache注解即可实现缓存逻辑透明化，让开发者关注业务逻辑。
+ 当事务回滚时，缓存也会自动回滚。
+ 支持比较复杂的缓存逻辑。
+ 提供缓存编程的一致性抽象，方便代码维护

需要注意的是`Spring Cache`并不针对多进程的应用环境进行专门的处理，也就是说，当应用程序处于分布式或者集群环境下时，需要针对具体的缓存进行相应的配置。

### Spring Cache 的注解
1. @Cacheable  
   @Cacheable是最主要的注解，它指定了被注解方法的返回值是可被缓存的。其工作原理是Spring首先在缓存中查找数据，如果没有则执行方法并缓存结果，然后返回数据。缓存名是必须提供的，可以使用引号、value或者 cacheNames 属性来定义名称。
   ```
    @Cacheable ("users")
    //Spring 3.x
    @Cacheable (value="users")
    //Spring 4.0新增了valve的别名cacheNames，比value 命名更达意，推荐使用
    @Cacheable(cacheNames)
   ```

	键生成器
	缓存的本质就是键/值对集合。在默认情况下，缓存抽象使用方法签名及参数值作为一个键值，并将该键与方法调用的结果组成键/值对。如果在Cache注解上没有指定key,则Spring会使用	KeyGenerator来生成一个key。Sping默认提供了 SimplekeyGenerator 生成器。其生成规则如下:
	1. 如果方法没有入参，则使用 `Simplekey.EMPTY` 作为key。
	2. 如果只有一个入参，则使用该入参作为key。
	3. 如果有多个入参，则返回包含所有入参的一个Simplekey。

	@Cacheable注解提供了实现该功能的key属性，通过该属性，可以使用SpEL指定自定义键：
	`@Cacheable(cacheNames="users",key="#user.userId")`

	带条件的缓存
	使用@Cacheable注解的condition屆性可按条件进行缓存，condition属性使用了SpEL表达式动态评估方法入参是否满足缓存条件。

2. @CachePut  
	@CachePut注解与@Cacheable注解效果几乎一样，它首先执行方法，然后将返回值放入缓存。  
	需要注意的是，在同一个方法内不能同时使用@CachePut和@Cacheable注解，因为它们拥有不同的特性。当@Cacheable注解跳过方法直接获取级存时，@CachePut注解会强制执行方法以更新缓存，这会导致意想不到的情况发生，如当注解都带入了条件屆性，就会使得它们彼此排斥。还需要注意的是，@CachePut注解的condition属性设置的缓存条件也不应该依赖于方法返回的结果(如condition="#result")，因为缓存条件是在方法执行前预先验证的。

3. @CacheEvict  
	@CacheEvict注解是@Cachable注解的反向操作，它负责从给定的缓存中移除一个值。大多数缓存框架都提供了缓存数据的有效期，使用该注解可以显式地从缓存中删除失效的缓存数据。该注解通常用于更新或者删除用户的操作。


4. 添加依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
    <version>2.4.3</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.4.3</version>
</dependency>
```

1. 配置类上加上 @EnableCaching 
2. Spring Boot will auto-configure a RedisCacheManager with default cache configuration. However, we can modify this configuration prior to cache manager initialization in a couple of useful ways.
```
@Bean
public RedisCacheConfiguration cacheConfiguration() {
    return RedisCacheConfiguration.defaultCacheConfig()
      .entryTtl(Duration.ofMinutes(60))
      .disableCachingNullValues()
      .serializeValuesWith(SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
}
```

```
@Bean
    public RedisCacheManagerBuilderCustomizer redisCacheManagerBuilderCustomizer() {
        RedisSerializationContext.SerializationPair jsoSerilizer = RedisSerializationContext
                .SerializationPair
                .fromSerializer(new GenericJackson2JsonRedisSerializer());

        return (builder) -> builder
                .withCacheConfiguration("sms",
                        RedisCacheConfiguration.defaultCacheConfig()
                                .entryTtl(Duration.ofMinutes(10))
                                .serializeKeysWith(jsoSerilizer)
                                .serializeValuesWith(jsoSerilizer))
                .withCacheConfiguration("customerCache",
                        RedisCacheConfiguration.defaultCacheConfig()
                                .entryTtl(Duration.ofMinutes(5))
                                .serializeKeysWith(jsoSerilizer)
                                .serializeValuesWith(jsoSerilizer));
    }
```



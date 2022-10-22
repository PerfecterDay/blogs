## Spring Cache的集成


1. 添加依赖
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

2. 配置类上加上 @EnableCaching 
3. Spring Boot will auto-configure a RedisCacheManager with default cache configuration. However, we can modify this configuration prior to cache manager initialization in a couple of useful ways.
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
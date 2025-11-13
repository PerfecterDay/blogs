# micrometer tracing
{docsify-updated}

> https://docs.micrometer.io/tracing/reference/index.html

## 概念

<center><img src="pics/trace-id.jpg" alt=""></center>


## Springboot 集成
```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-tracing-bom</artifactId>
            <version>${micrometer-tracing.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
      <groupId>io.micrometer</groupId>
      <artifactId>micrometer-tracing</artifactId>
    </dependency>
</dependencies>
```

logback配置：
```
%d{HH:mm:ss.SSS} [%thread] %-5level %X{traceId:-} %X{spanId:-} %logger{36} - %msg%n
```

### 跨线程传递 traceId/spanId :
```
@Bean
    public TaskDecorator taskDecorator() {
        return runnable -> ContextSnapshotFactory
                .builder()
                // you can limit what thread locals to propagate
                // (e.g. .captureKeyPredicate(ObservationThreadLocalAccessor.KEY::equals))
                .build()
                .captureAll()
                .wrap(runnable);
    }

    @Bean
    public ThreadPoolTaskExecutor executor(TaskDecorator taskDecorator) {
        return new ThreadPoolTaskExecutorBuilder()
                // add your own configs (e.g. maxPoolSize, threadNamePrefix)
                .taskDecorator(taskDecorator)
                .build();
    }
```

###
```

```
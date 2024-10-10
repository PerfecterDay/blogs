# Meters 种类
{docsify-updated}

## Counters
计数器只报告一个指标：计数。 计数器界面允许你以一个固定值递增，这个值必须是正数。

```
Normal rand = ...; // a random generator

MeterRegistry registry = ...
Counter counter = registry.counter("counter");

Flux.interval(Duration.ofMillis(10))
        .doOnEach(d -> {
            if (rand.nextDouble() + 0.1 > 0) {
                counter.increment();
            }
        })
        .blockLast();


Counter counter = Counter
    .builder("counter")
    .baseUnit("beans") // optional
    .description("a description of what this counter does") // optional
    .tags("region", "test") // optional
    .register(registry);
```

操作 Counters:
```
counter.increment();
counter.increment(10);
```

## Gauges
Gauges是一个用于获取当前值的句柄。 Gauges的典型示例包括集合或地图的大小，或处于运行状态的线程数量。将Gauges视为 "heisen-gauge"：一种只有在被观察到时才会发生变化的Gauges。 所有其他类型的仪表都会累积中间计数，然后将数据发送到度量后台。
```
List<String> list = registry.gauge("listGauge", Collections.emptyList(), new ArrayList<>(), List::size);
List<String> list2 = registry.gaugeCollectionSize("listSize2", Tags.empty(), new ArrayList<>());
Map<String, Integer> map = registry.gaugeMapSize("mapGauge", Tags.empty(), new HashMap<>());

Gauge gauge = Gauge
    .builder("gauge", myObj, myObj::gaugeValue)
    .description("a description of what this gauge does") // optional
    .tags("region", "test") // optional
    .register(registry);
```

一般来说，Gauge是自动获取的属性值，但是特殊情况下也可以手动操作 Gauge:
```
myGauge.set(27);
myGauge.set(11);
```

## Timers
计时器用于测量耗时和此类事件的频率。 定时器的所有实现都至少将耗时和事件计数报告为单独的时间序列，但也可以报告其他时间序列，具体取决于后端支持什么（最大值、百分位数、直方图）。   
典型的应用就是用来统计请求的处理时间。
```
Timer timer = Timer
    .builder("my.timer")
    .description("a description of what this timer does") // optional
    .tags("region", "test") // optional
    .register(registry);


SimpleMeterRegistry registry = new SimpleMeterRegistry();
Timer timer = registry.timer("app.event");
timer.record(() -> {
    try {
        TimeUnit.MILLISECONDS.sleep(15);
    } catch (InterruptedException ignored) {
    }
    });

timer.record(30, TimeUnit.MILLISECONDS);


Timer.Sample sample = Timer.start(registry);

// do stuff
Response response = ...

sample.stop(registry.timer("my.timer", "response", response.status()));
```

### `@Timed`注解
micrometer-core 模块包含一个 @Timed 注解，框架可使用该注解为特定类型的方法（例如为网络请求端点提供服务的方法）或所有方法添加定时支持。
首先需要注册一个 AspectJ 切面：
```
@Configuration
public class TimedConfiguration {
   @Bean
   public TimedAspect timedAspect(MeterRegistry registry) {
      return new TimedAspect(registry);
   }
}
```

然后在需要统计耗时的方法上加上 `@Timed` 注解即可：
```
@Service
public class ExampleService {

  @Timed
  public void sync() {
    // @Timed will record the execution time of this method,
    // from the start and until it exits normally or exceptionally.
    ...
  }

  @Async
  @Timed
  public CompletableFuture<?> async() {
    // @Timed will record the execution time of this method,
    // from the start and until the returned CompletableFuture
    // completes normally or exceptionally.
    return CompletableFuture.supplyAsync(...);
  }

}
```

## Distribution Summaries
分布汇总跟踪事件的分布。 它在结构上类似于计时器，但记录的值不代表时间单位。 例如，您可以使用分布汇总来测量访问服务器的请求的有效载荷大小。

```
DistributionSummary summary = registry.summary("response.size");

DistributionSummary summary = DistributionSummary
    .builder("response.size")
    .description("a description of what this summary does") // optional
    .baseUnit("bytes") // optional
    .tags("region", "test") // optional
    .scale(100) // optional
    .register(registry);
```

可以选择提供一个缩放因子(scale)，在记录每个样本时将其乘以该因子。

Micrometer 预选的百分位数直方图桶都是 1 到 `Long.MAX_VALUE` 之间的整数。 目前， `minimumExpectedValue` 和 `maximumExpectedValue` 用于控制水桶集的核心数量。 如果检测到您的最小/最大值范围较小，并根据您的摘要范围缩放预选水桶域，那么我们就没有其他控制水桶数量的工具了。

## Long Task Timers
长任务定时器是一种特殊的定时器，可以在被测事件仍在运行时测量时间。 普通定时器只记录任务完成后的持续时间。
```
@Timed(value = "aws.scrape", longTask = true)
@Scheduled(fixedDelay = 360000)
void scrapeResources() {
    // find instances, volumes, auto-scaling groups, etc...
}

LongTaskTimer scrapeTimer = registry.more().longTaskTimer("scrape");
void scrapeResources() {
    scrapeTimer.record(() => {
        // find instances, volumes, auto-scaling groups, etc...
    });
}

LongTaskTimer longTaskTimer = LongTaskTimer
    .builder("long.task.timer")
    .description("a description of what this timer does") // optional
    .tags("region", "test") // optional
    .register(registry);
```
## Histograms and Percentiles
```
Timer.builder("my.timer")
   .publishPercentiles(0.5, 0.95) // median and 95th percentile
   .publishPercentileHistogram()
   .serviceLevelObjectives(Duration.ofMillis(100))
   .minimumExpectedValue(Duration.ofMillis(1))
   .maximumExpectedValue(Duration.ofSeconds(10))
```


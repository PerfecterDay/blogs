# Meters 种类
{docisfy-updated}

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
```

## Distribution Summaries
## Long Task Timers
## Histograms and Percentiles

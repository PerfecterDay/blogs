# java基础-经典的垃圾收集器
{docsify-updated}

<center>
<img src="pics/gc-collector.png" alt="" width=40% height=40%>
</center>

图3-6展示了七种作用于不同分代的收集器，如果两个收集器之间存在连线，就说明它们可以搭配使用，图中收集器所处的区域，则表示它是属于新生代收集器抑或是老年代收集。

## GC选择指南
除非您的应用程序有相当严格的暂停时间要求，否则请首先运行您的应用程序，让虚拟机选择一个收集器。

如有必要，调整堆大小以提高性能。如果性能仍然达不到您的目标，那么请使用以下指南作为选择收集器的起点：

+ 如果应用程序的数据集很小（最多约 100 MB），则使用选项 `-XX:+UseSerialGC` 选择串行收集器。
+ 如果应用程序将在单个处理器上运行，且没有暂停时间要求，则选择串行采集器，选项为 `-XX:+UseSerialGC` 。
+ 如果
  + 应用程序的峰值性能是第一优先级，
  + 没有暂停时间要求或暂停时间为一秒或更长是可以接受的
  则让虚拟机选择收集器或使用 `-XX:+UseParallelGC` 选择并行收集器。
+ 如果响应时间比总体吞吐量更重要，而且垃圾收集暂停时间必须保持在大约一秒以内，那么请使用 `-XX:+UseG1GC` 或 `-XX:+UseConcMarkSweepGC` 选择主要并发的收集器。
+ 如果响应时间是重中之重，和/或您正在使用一个非常大的堆，那么请使用 `-XX:+UnlockExperimentalVMOptions -XX:+UseZGC` 选择一个完全并发的收集器。

这些指导原则只是选择收集器的起点，因为性能取决于堆的大小、应用程序维护的实时数据量以及可用处理器的数量和速度。

如果推荐的收集器不能达到预期性能，那么首先尝试调整堆和生成大小，以达到预期目标。如果性能仍然不佳，则尝试使用其他收集器：使用并发收集器减少暂停时间，使用并行收集器提高多处理器硬件的总体吞吐量。


+ `-XX:ParallelGCThreads=<N>`: 并发执行GC的线程数
+ `-XX:MaxGCPauseMillis=<N>` : 最大停顿时间
+ `-XX:GCTimeRatio=<N>` : GC时间和应用运行时间的比率


## GC日志配置
```
# 打印基本 GC 信息
-XX:+PrintGCDetails  ---> -Xlog:gc*
-XX:+PrintGCDateStamps  ---> -Xlog:gc*::time

# 打印对象分布
-XX:+PrintTenuringDistribution 

# 打印堆数据
-XX:+PrintHeapAtGC 

# 打印Reference处理信息
-XX:+PrintReferenceGC 

# 打印STW时间
-XX:+PrintGCApplicationStoppedTime

# 打印safepoint信息
-XX:+PrintSafepointStatistics 
-XX:PrintSafepointStatisticsCount=1

# GC日志输出的文件路径
-Xloggc:/path/to/gc-%t.log ---> -Xlog:gc*:gc.log
# 开启日志文件分割
-XX:+UseGCLogFileRotation 
# 最多分割几个文件，超过之后从头文件开始写
-XX:NumberOfGCLogFiles=14
# 每个文件上限大小，超过就触发分割
-XX:GCLogFileSize=100M
```

JDK 9 之后可以使用
```
-Xlog:gc*=trace:file="gc.log":tags,time,uptime,level
```

## 参数
`java -XX:+PrintFlagsFinal -version` 可以查看所有的支持的VM参数设置

-XX:CICompilerCount=2 -XX:InitialHeapSize=127926272 -XX:MaxHeapSize=2030043136 -XX:MaxNewSize=676659200 -XX:MinHeapDeltaBytes=196608 -XX:NewSize=42598400 -XX:NonNMethodCodeHeapSize=5824844 -XX:NonProfiledCodeHeapSize=122916698 -XX:OldSize=85327872 -XX:ProfiledCodeHeapSize=122916698 -XX:ReservedCodeCacheSize=251658240 -XX:+SegmentedCodeCache -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:+UseSerialGC 
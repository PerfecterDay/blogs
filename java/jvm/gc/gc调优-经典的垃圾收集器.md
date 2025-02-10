# java基础-经典的垃圾收集器
{docsify-updated}

<center>
<img src="pics/gc-collector.png" alt="" width=40% height=40%>
</center>

图3-6展示了七种作用于不同分代的收集器，如果两个收集器之间存在连线，就说明它们可以搭配使用，图中收集器所处的区域，则表示它是属于新生代收集器抑或是老年代收集。

## 默认的 JVM 参数
这些是重要的垃圾收集器、堆大小和运行时编译器默认选择：
+ 垃圾优先（G1）收集器
+ GC线程的最大数量受堆大小和可用CPU资源的限制 (-XX:ConcGCThreads)
+ 物理内存1/64的初始堆大小 (-XX:InitialHeapSize)
+ 物理内存的1/4的最大堆大小 (-XX:MaxHeapSize)
+ 分层编译器，使用C1和C2


## GC 优化的目标
Java HotSpot VM垃圾收集器可以配置为优先满足两个目标之一：
+ 最大暂停时间
+ 应用程序吞吐量。

如果达到首选目标，GC将尝试将另一个目标最大化。当然，这些目标并不总是可以实现的：应用程序需要最小堆来保存至少所有实时数据，其他配置可能会阻止实现部分或所有期望的目标。

### 最大暂停时间目标
暂停时间是垃圾收集器停止应用程序并回收垃圾空间的持续时间。最大暂停时间目标的是限制最大暂停时间到一个限定值。

GC会保存暂停的平均时间和方差。平均值是从执行开始时开始的，但它是加权的，最近的停顿时间权重更高。如果平均加暂停时间的方差大于最大暂停时间目标，则垃圾收集器认为目标没有实现。

最大暂停时间目标由命令行选项指定 `-XX:MaxGCPauseMillis=<nnn>` 。这会暗示GC我们需要`<nnn>`毫秒或更短的暂停时间。垃圾收集器会调整Java堆大小和与垃圾收集相关的其他参数，试图保持垃圾收集暂停短于`<nnn>`毫秒。最大暂停时间目标的默认值因收集器而异。这些调整可能会导致垃圾收集更频繁，从而降低应用程序的整体吞吐量。在某些情况下，无法实现所需的暂停时间目标。

通常来说，要想降低停顿时间，需要缩小堆，这样GC扫描和回收的空间就降低，相当于减少了GC的工作量。

### 吞吐量目标
吞吐量目标是用收集垃圾的时间来衡量的，在垃圾收集之外花费的时间是应用运行的时间。

目标由命令行选项 `-XX:GCTimeRatio=nnn` 指定。垃圾收集时间与应用时间的比率为`1/(1+nnn)`。例如， `-XX:GCTimeRatio=19` 设定的目标是垃圾收集时间占总时间的1/20(5%)。

在垃圾收集中花费的时间是所有垃圾收集诱导的暂停的总时间。如果吞吐量目标未实现，那么垃圾收集器的一个可能操作是增加堆的大小，这样运行GC的间隔时间会更长。

如果满足了吞吐量和最大暂停时间目标，那么垃圾收集器会减少堆的大小，直到无法实现其中一个目标（总是吞吐量目标）。垃圾收集器可以使用的最小和最大堆大小可以分别使用`-Xms=<nnn>` 和 `-Xmx=<mmm>` 设置最小和最大堆大小。

通常来说，要想提高吞吐量，需要增大堆，这样应用可使用的内存就会更多，GC次数会更少，应用运行的时间就会更长。

## 分代理论（UseSerialGC为例）
<center><img src="pics/gc-generation.png" alt=""></center>

启动时，Java HotSpot VM在地址空间中保留整个Java堆，但除非需要，否则不会为其分配任何物理内存。覆盖Java堆的整个地址空间在逻辑上分为年轻一代和老一代。为对象内存保留的完整地址空间可以分为年轻一代和老一代。年轻一代由eden和两个幸存者空间组成。大多数对象最初在eden中分配。一个幸存者空间在任何时候都是空的，在垃圾收集期间作为eden和另一个幸存者空间的活体对象的目的地；垃圾收集后，eden和源幸存者空间是空的。在下一个垃圾收集中，交换两个幸存者空间的目的。最近填充的一个空间是活体物体的来源，这些物体被复制到另一个幸存者空间中。对象以这种方式在幸存者空间之间复制，直到它们被复制了一定次数，或者那里没有足够的空间。这些对象被复制到旧区域。这个过程也被称为衰老。

### 堆大小
+ `-XX:MinHeapFreeRatio`: 空闲空间的最小比率，小于这个值，堆会扩展
+ `-XX:MaxHeapFreeRatio`: 空闲空间的最大比率，大于这个值，堆会缩小
+ `-Xms`: 堆最小值
+ `-Xmx`: 堆最大值

`-XX:MinHeapFreeRatio:40 -XX:MaxHeapFreeRatio:70 -Xms:1G -Xmx:4G` 表示如果某一代的空闲空间百分比低于40%，那么该代将扩展，以保持40%的空闲空间，直到达到该代允许的最大大小 4G。类似地，如果空闲空间超过70%，那么该代将收缩，以使只有70%的空间是空闲的，但要不会小于该代的最小大小限制 1G。

### 年轻代
+ `–XX:NewRatio` : 代表老年代与年轻代的比率大小。`-XX:NewRatio=3` 代表老年代：年轻代=3，年轻代是总堆大小的 1/4 。
+ `-XX:NewSize` : 年轻代大小的最小值
+ `-XX:MaxNewSize`: 年轻代大小的最大值
+ `-XX:SurvivorRatio`: `-XX:SurvivorRatio=6`代表 eden:survivor=1:6，有两个 survivor ，所以单个 survivor 占年轻代的1/8


## GC日志配置
```
# 打印基本 GC 信息
-XX:+PrintGCDetails  ---> -Xlog:gc*
-XX:+PrintGCDateStamps  ---> -Xlog:gc*::time
-verbose:gc

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



-XX:CICompilerCount=4 -XX:ConcGCThreads=2 -XX:G1ConcRefinementThreads=8 -XX:G1HeapRegionSize=1048576 -XX:GCDrainStackTargetSize=64 -XX:InitialHeapSize=257949696 -XX:MarkStackSize=4194304 -XX:MaxHeapSize=4127195136 -XX:MaxNewSize=2475687936 -XX:MinHeapDeltaBytes=1048576 -XX:NonNMethodCodeHeapSize=5835340 -XX:NonProfiledCodeHeapSize=122911450 -XX:ProfiledCodeHeapSize=122911450 -XX:ReservedCodeCacheSize=251658240 -XX:+SegmentedCodeCache -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:+UseG1GC 


```
-XX:MaxDirectmemorySize :设置直接内存容量，如果不指定，默认64M。
-XX:CICompilerCount=3 
-XX:ConcGCThreads=1 
-XX:G1ConcRefinementThreads=4 
-XX:G1HeapRegionSize=1048576 
-XX:GCDrainStackTargetSize=64 
-XX:InitialHeapSize=130023424 
-XX:MarkStackSize=4194304 
-XX:MaxHeapSize=2051014656 
-XX:MaxNewSize=1229979648 
-XX:MinHeapDeltaBytes=1048576 
-XX:NonNMethodCodeHeapSize=5830092 
-XX:NonProfiledCodeHeapSize=122914074 
-XX:ProfiledCodeHeapSize=122914074 
-XX:ReservedCodeCacheSize=251658240 
-XX:+SegmentedCodeCache 
-XX:+UseCompressedClassPointers 
-XX:+UseCompressedOops -XX:+UseG1GC
```
## 可选的垃圾收集器

### Serial Collector
串行GC使用单个线程来执行所有垃圾收集工作，这使得它相对高效，因为线程之间没有通信开销。它最适合单处理器机器，因为它无法利用多处理器硬件，它在多处理器上对具有小数据集（高达约100 MB）的应用程序也很有用。默认情况下，在某些硬件和操作系统配置上选择串行收集器，或者可以使用`-XX:+UseSerialGC`选项显式启用。

### Parallel Collector
并行收集器也被称为吞吐量收集器，它是一个类似于串行收集器的代号收集器。串行收集器和并行收集器之间的主要区别在于，并行收集器具有多个线程，用于加快垃圾收集速度。并行收集器适用于在多处理器或多线程硬件上运行的中型到大型数据集的应用程序。您可以使用`-XX:+UseParallelGC`选项启用它。
并行压缩是一种功能，它使并行收集器能够并行执行 major GC。如果没有并行压缩，major GC 使用单个线程执行，这可以显著限制可扩展性。如果指定了选项`-XX:+UseParallelGC`，则默认启用并行压缩。您可以使用 `-XX:-UseParallelOldGC` 选项禁用它。

### The Mostly Concurrent Collectors
Concurrent Mark Sweep (CMS) and Garbage-First (G1) are the two mostly concurrent collectors.
+ `-XX:+UseConcMarkSweepGC` 使用 CMS
+ `-XX:+UseG1GC` 使用 G1

JDK9中淘汰了 CMS 。

### ZGC
Z垃圾收集器（ZGC）是一个可扩展的低延迟垃圾收集器。ZGC并发地执行所有昂贵的工作，而不会停止应用程序线程的执行。

ZGC适用于需要低延迟（少于10毫秒的暂停）和/或使用非常大的堆（多兆字节）的应用程序。可以使用 `-XX:+UseZGC` 选项来启用ZGC。

### GC选择指南
除非您的应用程序有相当严格的暂停时间要求，否则请首先运行您的应用程序，让虚拟机选择一个收集器。如有必要，调整堆大小以提高性能。  
如果性能仍然达不到您的目标，那么请使用以下指南作为选择收集器的起点：

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
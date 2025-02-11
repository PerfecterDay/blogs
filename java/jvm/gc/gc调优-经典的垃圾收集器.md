# java基础-经典的垃圾收集器
{docsify-updated}

<center>
<img src="pics/gc-collector.png" alt="" width=40% height=40%>
</center>

图3-6展示了七种作用于不同分代的收集器，如果两个收集器之间存在连线，就说明它们可以搭配使用，图中收集器所处的区域，则表示它是属于新生代收集器抑或是老年代收集。

## 分代理论（UseSerialGC为例）
<center><img src="pics/gc-generation.png" alt=""></center>

启动时，Java HotSpot VM在地址空间中保留整个Java堆，但除非需要，否则不会为其分配任何物理内存。覆盖Java堆的整个地址空间在逻辑上分为年轻一代和老一代。为对象内存保留的完整地址空间可以分为年轻一代和老一代。年轻一代由eden和两个幸存者空间组成。大多数对象最初在eden中分配。一个幸存者空间在任何时候都是空的，在垃圾收集期间作为eden和另一个幸存者空间的活体对象的目的地；垃圾收集后，eden和源幸存者空间是空的。在下一个垃圾收集中，交换两个幸存者空间的目的。最近填充的一个空间是活体物体的来源，这些物体被复制到另一个幸存者空间中。对象以这种方式在幸存者空间之间复制，直到它们被复制了一定次数，或者那里没有足够的空间。这些对象被复制到旧区域。这个过程也被称为衰老。


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

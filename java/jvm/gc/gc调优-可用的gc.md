# 可选的垃圾收集器
{docsify-updated}

## Serial Collector
串行GC使用单个线程来执行所有垃圾收集工作，这使得它相对高效，因为线程之间没有通信开销。它最适合单处理器机器，因为它无法利用多处理器硬件，它在多处理器上对具有小数据集（高达约100 MB）的应用程序也很有用。默认情况下，在某些硬件和操作系统配置上选择串行收集器，或者可以使用`-XX:+UseSerialGC`选项显式启用。

## Parallel Collector
并行收集器也被称为吞吐量收集器，它是一个类似于串行收集器的代号收集器。串行收集器和并行收集器之间的主要区别在于，并行收集器具有多个线程，用于加快垃圾收集速度。并行收集器适用于在多处理器或多线程硬件上运行的中型到大型数据集的应用程序。您可以使用`-XX:+UseParallelGC`选项启用它。
并行压缩是一种功能，它使并行收集器能够并行执行 major GC。如果没有并行压缩，major GC 使用单个线程执行，这可以显著限制可扩展性。如果指定了选项`-XX:+UseParallelGC`，则默认启用并行压缩。您可以使用 `-XX:-UseParallelOldGC` 选项禁用它。


## Garbage-First (G1) Garbage Collector
G1 主要是并发收集器。 大部分并发收集器与应用程序同时执行一些昂贵的工作。 这种收集器设计用于从小型机扩展到具有大量内存的大型多处理器机。 它能在实现高吞吐量的同时达到暂停时间目标。

大多数硬件和操作系统配置默认选择 G1，也可使用 `-XX:+UseG1GC` 显式启用 G1。

## ZGC
ZGC 的最大暂停时间不超过一毫秒，但会牺牲一些吞吐量。 它适用于需要低延迟的应用。 暂停时间与所使用的堆大小无关。 ZGC 适用于几百兆字节到 16TB 的堆大小，要启用此功能，请使用 `-XX:+UseZGC` 选项。

默认使用 ZGC 的分代版本，ZGC 的非分代版本已被弃用。 不过，要恢复使用以前的非分代版本 ZGC，除了使用 `-XX:+UseZGC` 选项外，还要使用 `-XX:-ZGenerational` 选项。

## GC选择指南
除非您的应用程序有相当严格的暂停时间要求，否则请首先运行您的应用程序，让虚拟机自动选择一个收集器。如有必要，调整堆大小以提高性能。如果性能仍然达不到您的目标，那么请使用以下指南作为选择收集器的起点：

+ 如果应用程序的数据集很小（最多约 100 MB），则使用选项 `-XX:+UseSerialGC` 选择串行收集器。
+ 如果应用程序将在单个处理器上运行，且没有暂停时间要求，则选择串行采集器，选项为 `-XX:+UseSerialGC` 。
+ 如果
  + 应用程序的峰值性能是第一优先级，
  + 没有暂停时间要求或暂停时间为一秒或更长是可以接受的
  则让虚拟机选择收集器或使用 `-XX:+UseParallelGC` 选择并行收集器。
+ 如果响应时间比总体吞吐量更重要，而且垃圾收集暂停时间必须保持在大约一秒以内，那么请使用 `-XX:+UseG1GC` 选择G1垃圾收集器。
+ 如果响应时间是重中之重，和/或您正在使用一个非常大的堆，那么请使用 `-XX:+UnlockExperimentalVMOptions -XX:+UseZGC` 选择一个完全并发的收集器。

这些指导原则只是选择收集器的起点，因为性能取决于堆的大小、应用程序维护的实时数据量以及可用处理器的数量和速度。

如果推荐的收集器不能达到预期性能，那么：
1. 首先尝试调整堆和分代的大小，以达到预期目标。
2. 如果性能仍然不佳，则尝试使用其他收集器：使用并发收集器减少暂停时间，使用并行收集器提高多处理器硬件的总体吞吐量。
# 线程池
{docsify-updated}

## 线程池的好处

合理利用线程池能够带来三个好处：

1. 降低资源消耗。通过重复利用已创建的线程降低线程**创建和销毁造成的消耗**。
2. 提高响应速度。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
3. 提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源（频繁切换调度开销），还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

但是要做到合理的利用线程池，必须对其原理了如指掌。

## 线程池类图结构
<center><img src="pics/threadpool1.jpg" width=40% height=40%></center>

`AbstractExecutorService` 是 `ExecutorService` 的继承类。

## 内置线程池
JDK1.5 提供了一个 Executors 工厂来生产线程池，该工厂里包含如下几个静态工厂方法来创建线程池：
1. `newCachedThreadPool()`: 创建一个具有缓存功能的线程池，系统根据需要创建线程，这些线程将会被缓存到线程池中
2. `newFixedThreadPool(int nThreads)`: 创建一个可重用的、具有固定线程数的线程池
3. `newSingleThreadExecutor()`: 创建一个只有单线程的线程池
4. `newScheduledThreadPool(int corePoolSize )`: 创建一个具有固定线程数的线程池，它可以在指定延迟后执行线程任务。corePoolSize指池中保存的线程数，即使线程是空闲的也保存下来
5. `newSingleThreadScheduledExecutor()`: 创建一个只有单线程的线程池，它可以在指定延迟后执行线程任务。

## 自创建线程池

我们可以通过继承 `ThreadPoolExecutor` 来创建一个线程池。

    public class ThreadPoolExecutor extends AbstractExecutorService {
        .....
        public ThreadPoolExecutor(int corePoolSize,
                                      int maximumPoolSize,
                                      long keepAliveTime,
                                      TimeUnit unit,
                                      BlockingQueue<Runnable> workQueue,
                                      ThreadFactory threadFactory,
                                      RejectedExecutionHandler handler) {
                if (corePoolSize < 0 ||
                    maximumPoolSize <= 0 ||
                    maximumPoolSize < corePoolSize ||
                    keepAliveTime < 0)
                    throw new IllegalArgumentException();
                if (workQueue == null || threadFactory == null || handler == null)
                    throw new NullPointerException();
                this.corePoolSize = corePoolSize;
                this.maximumPoolSize = maximumPoolSize;
                this.workQueue = workQueue;
                this.keepAliveTime = unit.toNanos(keepAliveTime);
                this.threadFactory = threadFactory;
                this.handler = handler;
            }
    }
    

```
 public ThreadPoolExecutor(int corePoolSize,
                                      int maximumPoolSize,
                                      long keepAliveTime,
                                      TimeUnit unit,
                                      BlockingQueue<Runnable> workQueue,
                                      ThreadFactory threadFactory,
                                      RejectedExecutionHandler handler)
```   

上面这个是它的核心构造函数，创建一个线程池需要输入几个参数：

1. **int corePoolSize**（线程池的基本大小）：当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的prestartAllCoreThreads方法，线程池会提前创建并启动所有基本线程。
2. **int maximumPoolSize**（线程池最大大小）：线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是如果使用了无界的任务队列这个参数就没什么效果。
3. **long keepAliveTime**（线程活动保持时间）：线程池的工作线程空闲后，保持存活的时间。所以如果任务很多，并且每个任务执行的时间比较短，可以调大这个时间，提高线程的利用率。
4. **TimeUnit unit**（线程活动保持时间的单位）：可选的单位有天（DAYS），小时（HOURS），分钟（MINUTES），毫秒(MILLISECONDS)，微秒(MICROSECONDS, 千分之一毫秒)和毫微秒(NANOSECONDS, 千分之一微秒)。
5. **BlockingQueue<Runnable> workQueue**（任务队列）：用于保存等待执行的任务的阻塞队列。可以选择以下几个阻塞队列：
    1. ArrayBlockingQueue：是一个基于数组结构的有界阻塞队列，此队列按 FIFO（先进先出）原则对元素进行排序。
    2. LinkedBlockingQueue：一个基于链表结构的阻塞队列，此队列按FIFO （先进先出） 排序元素，吞吐量通常要高于ArrayBlockingQueue。静态工厂方法Executors.newFixedThreadPool()使用了这个队列。
    3. SynchronousQueue：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue，静态工厂方法Executors.newCachedThreadPool使用了这个队列。
    4. PriorityBlockingQueue：一个具有优先级得无限阻塞队列。
6. **ThreadFactory threadFactory**：用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字，Debug和定位问题时非常又帮助。
7. **RejectedExecutionHandler handler**（饱和策略）：当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。以下是JDK1.5提供的四种策略:
    1. AbortPolicy：直接抛出异常。
    2. CallerRunsPolicy：只用调用者所在线程来运行任务。
    3. DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。
    4. DiscardPolicy：不处理，丢弃掉。
当然也可以根据应用场景需要来实现RejectedExecutionHandler接口自定义策略。如记录日志或持久化不能处理的任务。

```
REGS 线程池配置
corePoolSize - 10
maximumPoolSize - 50
ArrayBlockingQueue(500)
keepAliveTime - 600000L(ms) -> 600s -> 10分钟
```

### 向线程池提交任务
我们可以使用`execute`提交的任务，但是`execute`方法没有返回值，所以无法判断任务知否被线程池执行成功。通过以下代码可知`execute`方法输入的任务是一个Runnable类的实例。
    
    void execute(Runnable command);
我们也可以使用 `submit` 方法来提交任务，它会返回一个`future`,那么我们可以通过这个`future`来判断任务是否执行成功，通过`future`的`get`方法来获取返回值，`get`方法会阻塞住直到任务完成，而使用`get(long timeout, TimeUnit unit)`方法则会阻塞一段时间后立即返回，这时有可能任务没有执行完。

    <T> Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);//任务执行完后，返回 result 结果

<center><img src="pics/how-to-use-executor.jpg" width=40% hright=40%></center>

### 线程池的关闭
我们可以通过调用线程池的`shutdown`或`shutdownNow`方法来关闭线程池，但是它们的实现原理不同：

1. shutdown的原理是只是将线程池的状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线程。
2. shutdownNow的原理是遍历线程池中的工作线程，然后逐个调用线程的 `interrupt` 方法来中断线程，所以无法响应中断的任务可能永远无法终止。shutdownNow会首先将线程池的状态设置成STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表。

只要调用了这两个关闭方法的其中一个，`isShutdown`方法就会返回true。当所有的任务都已关闭后,才表示线程池关闭成功，这时调用`isTerminaed`方法会返回true。至于我们应该调用哪一种方法来关闭线程池，应该由提交到线程池的任务特性决定，通常调用shutdown来关闭线程池，如果任务不一定要执行完，则可以调用`shutdownNow`。

## 线程池的分析
当提交一个新任务到线程池时，线程池的处理流程如下：

1. 首先线程池判断基本线程池是否已满？没满（线程数小于核心线程数），即使有空闲线程，也会创建一个工作线程来执行任务。满了（已经达到核心线程数）则进入下个流程。
2. 看看是否有空闲线程可用，有空闲线程则使用空闲线程执行任务；若无空闲线程，则进入下个流程。
3. 线程池判断工作队列是否已满？没满，则将新提交的任务存储在工作队列里。满了，则进入下个流程。
1. 最后线程池判断整个线程池是否已满？没满，则创建一个新的工作线程来执行任务，满了，则交给饱和策略来处理这个任务。
如下图所示：

<center><img src="pics/Java线程池主要工作流程.jpg" alt=""></center>
 

注意：上述第二步 --> 看看是否有空闲线程可用，有空闲线程则使用空闲线程执行任务；若无空闲线程，则进入下个流程。没有在图中体现出来。

### 源码分析

    public void execute(Runnable command) {
        if (command == null)
                throw new NullPointerException();
        
            int c = ctl.get();
            //如果线程数小于核心线程数，则创建线程并执行当前任务
            if (workerCountOf(c) < corePoolSize) {
                if (addWorker(command, true))
                    return;
                c = ctl.get();
            }
            //如线程数大于等于基本线程数或线程创建失败，则将当前任务放到工作队列中。
            if (isRunning(c) && workQueue.offer(command)) {
                int recheck = ctl.get();
                if (! isRunning(recheck) && remove(command))
                    reject(command);
                else if (workerCountOf(recheck) == 0)
                    addWorker(null, false);
            }
            //拒绝任务
            else if (!addWorker(command, false))
                reject(command);
    }


## 合理配置线程池

合理的配置线程池
要想合理的配置线程池，就必须首先分析任务特性，可以从以下几个角度来进行分析：
1. 任务的性质：CPU密集型任务，IO密集型任务和混合型任务。
2. 任务的优先级：高，中和低。
3. 任务的执行时间：长，中和短。
4. 任务的依赖性：是否依赖其他系统资源，如数据库连接。

任务性质不同的任务可以用不同规模的线程池分开处理。
+ CPU密集型任务配置尽可能少的线程数量，如配置*N<sub>cpu</sub>+1*个线程的线程池。因为如果CPU密集型的任务在执行时，会占用CPU较多时间，增加很多线程也无法提高CPU利用率。
+ IO密集型任务则由于需要等待IO操作，线程并不是一直在执行任务，可能要等待IO操作，为了避免CPU闲置，则配置尽可能多的线程以提高CPU的利用率，如*2N<sub>cpu</sub>*。
+ 混合型的任务，如果可以拆分，则将其拆分成一个CPU密集型任务和一个IO密集型任务，只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐率要高于串行执行的吞吐率，如果这两个任务执行时间相差太大，则没必要进行分解。

我们可以通过`Runtime.getRuntime().availableProcessors()`方法获得当前设备的CPU个数。

优先级不同的任务可以使用优先级队列`PriorityBlockingQueue`来处理。它可以让优先级高的任务先得到执行，需要注意的是如果一直有优先级高的任务提交到队列里，那么优先级低的任务可能永远不能执行。

执行时间不同的任务可以交给不同规模的线程池来处理，或者也可以使用优先级队列，让执行时间短的任务先执行。

依赖数据库连接池的任务，因为线程提交SQL后需要等待数据库返回结果，如果等待的时间越长CPU空闲时间就越长，那么线程数应该设置越大，这样才能更好的利用CPU。

建议使用有界队列，有界队列能增加系统的稳定性和预警能力，可以根据需要设大一点，比如几千。有一次我们组使用的后台任务线程池的队列和线程池全满了，不断的抛出抛弃任务的异常，通过排查发现是数据库出现了问题，导致执行SQL变得非常缓慢，因为后台任务线程池里的任务全是需要向数据库查询和插入数据的，所以导致线程池里的工作线程全部阻塞住，任务积压在线程池里。如果当时我们设置成无界队列，线程池的队列就会越来越多，有可能会撑满内存，导致整个系统不可用，而不只是后台任务出现问题。当然我们的系统所有的任务是用的单独的服务器部署的，而我们使用不同规模的线程池跑不同类型的任务，但是出现这样问题时也会影响到其他任务。


## 线程池的监控
通过线程池提供的参数进行监控。线程池里有一些属性在监控线程池的时候可以使用

+ taskCount：线程池需要执行的任务数量。
+ completedTaskCount：线程池在运行过程中已完成的任务数量。小于或等于taskCount。
+ largestPoolSize：线程池曾经创建过的最大线程数量。通过这个数据可以知道线程池是否满过。如等于线程池的最大大小，则表示线程池曾经满了。
+ getPoolSize:线程池的线程数量。如果线程池不销毁的话，池里的线程不会自动销毁，所以这个大小只增不减。
+ getActiveCount：获取活动的线程数。

通过扩展线程池进行监控。通过继承线程池并重写线程池的 `beforeExecute` ， `afterExecute` 和 `terminated` 方法，我们可以在任务执行前，执行后和线程池关闭前干一些事情。如监控任务的平均执行时间，最大执行时间和最小执行时间等。这几个方法在线程池里是空方法。如：  
```protected void beforeExecute(Thread t, Runnable r) { }```

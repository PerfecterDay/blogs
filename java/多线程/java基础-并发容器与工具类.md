# Java 提供的并发容器与工具类
{docsify-updated}

## 并发容器
1. 阻塞队列  
   阻塞队列本身是线程安全的，而且具有协调线程的作用。当试图从空队列中取出元素或者往满队列中添加元素时，将会阻塞线程。上述描述更精确的可以说是，当阻塞队列中没有可以出队的元素时或者不能添加元素时将阻塞线程，如果满足了出队或入队条件会自动唤醒线程。
   <center><img src="pics/blocking-queue.png" width="50%"></center>
   
   如果要将队列用作线程同步工具，就要使用`put`和`take`方法。

   + `LinkedBlockkingQueue`：默认大小无边界，也可以指定最大容量
   + `LinkedBlockkingDeque`：双端队列版本
   + `ArrayBlockingQueue`: 构造时需要指定大小，并且可以提供参数来指定公平性，若设置为公平的，那么等待时间最长的线程会优先得到处理，这通常会降低性能。
   + `PriorityBlockingQueue`：优先级的阻塞队列。
   + `DelayQueue`：用来保存实现了`Delayed`接口的对象，`Delayed`接口有一个`public long getDelay(TimeUnit unit)`方法，只有这个方法返回负值时才能出队，没有元素能出队时，则阻塞消费线程。另外，元素还必须实现`public int compareTo(Delayed o)`接口用来排序元素。
2. 高效的映射表、集合和队列  
   java.util.current 包中提供了一些高效的并发容器类，这些集合使用复杂的算法，通过允许访问数据结构的不同部分来使竞争极小化。
   + `ConcurrentHashMap`
   + `ConcurrentSkipListMap`
   + `ConcurrentSkipListSet`
   + `ConcurrentLinkedQueue`
   + `CopyOnWriteArrayList`：当线程修改集合时，将对底层数组进行拷贝
   + `CopyOnWriteArraySet`：当线程修改集合时，将对底层数组进行拷贝
3. 同步包装器  
   使用 `Collections` 工具类的以 synchronized 开头的方法，能将普通集合包装成线程安全的集合。通常，不应该使用这种包装器包装的集合，而应该使用java.util.current 包中的集合。一个例外是，如果需要经常被修改的数组列表或集合（ArrayList/ArraySet），包装器集合比`CopyOnWriteArrayList`/`CopyOnWriteArraySet`更加高效。

## 并发工具类
1. CountDownLatch  
	假如有这样一个需求：我们需要解析一个Excel里多个sheet的数据，此时可以考虑使用多线程，每个线程解析一个sheet里的数据，等到所有的sheet都解析完之后，程序需要提示解析完成。在这个需求中，要实现主线程等待所有线程完成sheet的解析操作，最简单的做法是使用join()方法，在主线程中对所有的子线程调用其 join() 方法，表示主线程要等待所有的子线程完成才会继续执行。

	除此之外，我们也可以使用 `CountDownLatch` 类来实现此功能。 `CountDownLatch` 构造函数接收一个 int 类型的参数作为计数器，它会让一个线程集等待到计数变为0。如果你想等待N个线程完成，这里就传入N。

	当我们调用 `CountDownLatch` 的 `countDown` 方法时，N就会减1， `CountDownLatch` 的 `await` 方法会阻塞当前线程，直到N变成零。由于 `countDown` 方法可以用在任何地方，所以这里说的N个点，可以是N个线程，也可以是1个线程里的N个执行步骤。用在多个线程时，只需要把这个 `CountDownLatch` 的引用传递到线程里即可。

	```
	public class App {
		public static void main(String[] args) throws ExecutionException, InterruptedException {
			CountDownLatch countDownLatch = new CountDownLatch(3);
			//创建线程池
			ExecutorService executor = Executors.newCachedThreadPool();
			Future<Integer> future = executor.submit(new Task(countDownLatch,"1"));
			executor.submit(new Task(countDownLatch,"2"));
			executor.submit(new Task(countDownLatch,"3"));
			//这一步get会阻塞当前线程
			countDownLatch.await();
			System.out.println("所有的线程都执行完毕");
			executor.shutdown();
		}
	}

	class Task implements Callable<Integer> {
		private CountDownLatch latch;
		public String name;
		public Task(CountDownLatch latch,String name){
			this.latch = latch;
			this.name = name;
		}
		@Override
		public Integer call() throws Exception {
			System.out.println("子线程"+ name + "在进行计算");
			Thread.sleep(20000);
			latch.countDown();
			return 1;
		}

	}
	```

2. 同步屏障 CyclicBarrier  
	`CyclicBarrier` 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。

	`CyclicBarrier` 默认的构造方法是 `CyclicBarrier(int parties)`，其参数表示屏障拦截的线程数量，每个线程调用 `await` 方法告诉 `CyclicBarrier` 我已经到达了屏障，然后当前线程被阻塞。当指定数量的线程都到达屏障时，所有被阻塞的线程才会继续执行。

	`CyclicBarrier` 可以用于多线程计算数据，最后合并计算结果的场景。

	`CountDownLatch` 的计数器只能使用一次，而 `CyclicBarrier` 的计数器可以使用 `reset()` 方法重置。所以 `CyclicBarrier` 能处理更为复杂的业务场景。例如，如果计算发生错误，可以重置计数
	器，并让线程重新执行一次。

3. Semaphore  
	`Semaphore` （信号量）是用来控制同时访问特定资源的线程数量，它通过协调各个线程，以保证合理的使用公共资源。 `Semaphore` 的用法也很简单，首先线程使用 `Semaphore` 的 `acquire()` 方法获取一个许可证，使用完之后调用 `release()` 方法归还许可证。还可以用 `tryAcquire()` 方法尝试获取许可证。

4. 线程间交换数据的 Exchanger  
	`Exchanger` （交换者）是一个用于线程间协作的工具类。 `Exchanger` 用于进行线程间的数据交换。它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。这两个线程通过 `exchange` 方法交换数据，如果第一个线程先执行 `exchange()` 方法，它会一直等待第二个线程也执行 `exchange` 方法，当两个线程都到达同步点时，这两个线程就可以交换数据，将本线程生产出来的数据传递给对方。

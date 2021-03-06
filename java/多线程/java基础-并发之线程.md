# java线程
{docsify-updated}

### 线程的三种创建方式
1. 继承自 `Thread` 类，重写 `run()`方法，并调用 `start()` 方法启动线程
2. 实现 `Runnable` 接口，并在实例化 `Thread` 对象时，将接口实现对象作为构造参数传递进去
3. 创建 `Callable` 接口的实现类，实现 `call()` 方法，使用 `FutureTask` 对象来包装  `Callable` 对象，构造 `Thread` 时，将 `FutureTask`对象作为参数传递进去

三种方式的对比：
1. 继承 `Thread` 类后无法继承其它类，实现 `Runnable` 或 `Callable` 还可以继承其它类
2. `Callable` 接口与 `Runnable`接口相比，可以有返回值且可以抛出异常。

### 线程的一些属性
线程的常见属性包括：线程所属的线程组、线程优先级、是否是Daemon线程等。
可以调用 `Thread.setPriority()` 方法为线程设置优先级。
Daemon线程是一种支持型线程，因为它主要被用作程序中后台调度以及支持性工作。这意味着，当一个Java虚拟机中不存在非Daemon线程的时候，Java虚拟机将会退出。可以通过调用Thread.setDaemon(true)将线程设置为Daemon线程，必须在启动（调用 start）线程之前调用。
Daemon线程被用作完成支持性工作，但是在Java虚拟机退出时Daemon线程中的finally块并不一定会执行。

### 线程的状态
<center><img src="pics/thread-state.jpg" width=60% heght=60% /></center>

<center><img src="pics/thread-state-transform.png" width=60% heght=60% /></center>

注意：上图中的 WAITING/TIME_WAITING 状态不能直接转换到 RUNNABLE 状态，而是先转换到 BLOCKED 状态，获取到锁后才能转换到 RUNNABLE 状态。如果这时候正好没有其它线程在争取锁，那么被唤醒的线程可以直接获取锁进入 RUNNABLE 状态。

#### BLOCKED 和 WAITING 区别
0. BLOCKED 的线程等待的是一个**锁**，往往是在请求锁的时候，因为锁已经被别的线程持有，而**被动**(一般是由调度器将其设置为BLOCKED)的进入 BLOCKED 状态。
1. 线程通过调用`wait()`方法进入 WAITING 状态是一种主动行为，此时会释放持有的锁，一般是等待另一个线程的信号，用来完成线程的同步；而 BLOCKED 状态是被动的，线程希望继续执行，但是锁被别的线程获取，必须等待别的线程释放锁。
2. 站在调度器的角度上，假如一个线程释放了锁，调度器调度是需要考虑 BLOCKED 队列中的线程让它们争用锁，但是不需要考虑 WAITING 队列中的线程。
<center><img src="pics/wait-blocked.png" width=40% heght=40%></center>

## 线程之间的通信及同步

### volatile

关键字 volatile 可以用来修饰字段（成员变量），就是告知程序**任何对该变量的访问均需要从共享内存中获取，而对它的改变必须同步刷新回共享内存，它能保证所有线程对变量访问的可见性。**

### ThreadLocal
[ThreadLocal](./java基础-线程安全的实现方法.md#threadlocal)

### 等待通知机制
<center><img src="pics/wait-notify.jpg" width=50% heght=50%></center>

1. 调用 wait() 、notify() 或 notifyAll() 前需要先获取对象的锁
2. 调用 wait() 会立即释放对象锁（sleep()方法不会释放锁），线程由 RUNNING 变为 WAITING，并将当前线程放置到对象的等待队列
3. notify() 或 notifyAll() 调用后，等待线程依旧不会立即从 wait() 返回，需要调用 notify() 或 notifyAll()的线程释放锁之后，等待线程才有机会从 wait() 返回。
4. notify() 方法将等待队列中的一个等待线程从等待队列中移到同步队列中（阻塞于锁的队列），notifyAll() 则是将等待队列中的所有线程移到同步队列中，被移动的线程状态由 WAITING 变为 BLOCKED。
5. 从 wait() 方法返回的前提是获得对象的锁。

### 等待/通知的经典范式
等待方遵循如下原则：
1. 获取对象锁
2. 如果条件不满足，那么调用对象的wait()方法，被通知后仍要检查条件，因为被通知后不一定会立即获取到锁，其他线程有可能先获取锁并改变条件使得条件不再满足。
3. 条件满足则执行相应的逻辑。
对应的伪代码如下：
```
    synchronized(Object){
        while(条件不满足){
            Object.wait();
        }
        执行对应逻辑;
    }
```

通知方遵循如下原则：
1. 获取对象锁
2. 改变条件。
3. 通知所有等待在该对象上的线程。
对应的伪代码如下：
```
    synchronized(Object){
        改变条件;
        Object.notify();
    }
```


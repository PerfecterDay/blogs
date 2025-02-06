# Spring Lifecycle 与 SmartLifecycle
{docsify-updated}

在使用Spring的过程中，我们通常会用@PostConstruct和@PreDestroy在Bean初始化或销毁时执行一些操作，这些操作属于Bean声明周期级别的。

那么，就存在一些遗漏的场景，比如我们想在容器本身的生命周期（比如容器启动、停止）的事件上做一些工作，很典型的就是Spring Boot中启动内嵌的Web容器。该怎么办？

这就需要用到Spring提供的另外一个接口 `Lifecycle` 。这篇文件就介绍一下 `Lifecycle` 接口，以及比它更聪明（Smart）的 `SmartLifecycle` 。


## Lifecycel
```
public interface Lifecycle {
	void start();
	void stop();
	boolean isRunning();
}
```


## SmartLifecycle
```
public interface SmartLifecycle extends Lifecycle, Phased {
	int DEFAULT_PHASE = Integer.MAX_VALUE;

	default boolean isAutoStartup() {
		return true;
	}

	default void stop(Runnable callback) {
		stop();
		callback.run();
	}

	@Override
	default int getPhase() {
		return DEFAULT_PHASE;
	}

}
```

## 原理
<center><img src="pics/lifecycle2.png"></center>

从调用帧可以看出，`AbstractApplicationContext` 的 `finishRefresh()` 方法中会调用 `DefaultLifecycleProcessor` 的 `onRefresh()` 方法。

```
public void onRefresh() {
	startBeans(true);
	this.running = true;
}

public void start() {
	startBeans(false);
	this.running = true;
}
```

`onRefresh()` 方法调用 `startBeans()` 方法时，传入的是 `true` ；而 `start()` 调用 `startBeans()` 方法时，传入的是 `false` 。我们再看 `startBeans` 方法：

<center><img src="pics/lifecycle.png" width="80%"></center>

可以发现只有传入 `false` 参数或者 bean 是 `SmartLifecycle` 类型实例且是自动启动时，才会调用 `start()` 方法。所以，通过上面的 `DefaultLifecycleProcessor` 的 `start` 方法调用时(通过 `AbstractApplicationContext` 的 `start()`方法会调用)， `Lifecycle/SmartLifecycle` bean 会被同时调用；而 `onRefresh()` 方法只会调用 `SmartLifecycle` bean 的 `start` 方法。

这也就是为什么说需要显示调用 `ApplicationCOntext` 的 `start()` 方法， `Lifecycle` bean 才能生效的原因。
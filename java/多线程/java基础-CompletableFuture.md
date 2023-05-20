## Future/FutureTask 与 CompleteFuture
{docsify-updated}



>https://juejin.cn/post/6844904036299194382  
>https://juejin.cn/post/6844904033673560077

>https://java.jverson.com/


### Future
Future是为了配合Callable/Runnable而产生的，既然有返回值，那么返回什么？什么时候返回？这些问题其实都可以算在Future机制里。  
简单来讲我认为Future是一个句柄，即Callable任务返回给调用方这么一个句柄，通过这个句柄我们可以跟这个异步任务联系起来，我们可以通过future来对任务查询、取消、执行结果的获取，是调用方与异步执行方之间沟通的桥梁。

### CompletableFuture
```
//接受Runnable，无返回值，使用ForkJoinPool.commonPool()线程池
public static CompletableFuture<Void> runAsync(Runnable runnable)

//接受Runnable，无返回值，使用指定的executor线程池  
public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor)
  
//接受Supplier，返回U，使用ForkJoinPool.commonPool()线程池
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
  
//接受Supplier，返回U,使用指定的executor线程池 
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)
```


CompletableFuture<List<Xrate>> completeFXrate = CompletableFuture.supplyAsync(() ->
		tradeCenterService.getExchangeRate(user.getTradeAccount(), token)
);
CompletableFuture<List<Assets>> completeFAsset = CompletableFuture.supplyAsync(() ->
		tradeCenterService.getAssets(user.getTradeAccount(), token)
);
CompletableFuture<List<Stock>> completeFStock = CompletableFuture.supplyAsync(() ->
		tradeCenterService.getStocks(user.getTradeAccount(), token)
);
CompletableFuture.allOf(completeFXrate, completeFAsset, completeFStock)
		.join();

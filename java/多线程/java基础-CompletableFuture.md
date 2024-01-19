## Future/FutureTask 与 CompletableFuture
{docsify-updated}
>https://www.baeldung.com/java-completablefuture   

- [Future/FutureTask 与 CompletableFuture](#futurefuturetask-与-completablefuture)
  - [Future](#future)
  - [CompletableFuture](#completablefuture)
    - [将 CompletableFuture 当作简单Future来用](#将-completablefuture-当作简单future来用)
  - [相继执行多个任务](#相继执行多个任务)


异步计算很难推理。通常，我们希望将任何计算视为一系列步骤，但在异步计算中，表示为回调的操作往往分散在代码中，或者相互嵌套。当我们需要处理某个步骤中可能出现的错误时，情况就会变得更糟。

Java 5 中添加了 `Future` 接口来作为异步计算的结果，但它没有任何方法来组合这些计算或处理可能出现的错误。

Java 8 引入了 `CompletableFuture` 类。除了 `Future` 接口，它还实现了 `CompletionStage` 接口。该接口定义了异步计算步骤的契约，我们可以将其与其他步骤结合起来。

`CompletableFuture` 同时是一个构件和框架，拥有约 50 种不同的方法，用于组成、组合和执行异步计算步骤以及处理错误。

### Future
Future是为了配合Callable/Runnable而产生的，既然有返回值，那么返回什么？什么时候返回？这些问题其实都可以算在Future机制里。  
简单来讲我认为Future是一个句柄，即Callable任务返回给调用方这么一个句柄，通过这个句柄我们可以跟这个异步任务联系起来，我们可以通过future来对任务查询、取消、执行结果的获取，是调用方与异步执行方之间沟通的桥梁。

### CompletableFuture

#### 将 CompletableFuture 当作简单Future来用
首先，CompletableFuture 类实现了 Future 接口，因此我们可以将其用作 Future 实现，但需要附加完成逻辑。
例如，我们可以用一个无参数构造函数创建该类的一个实例来表示未来的某个结果，然后将其交给消费者，并在未来某个时间使用 complete 方法完成它。消费者可以使用 get 方法阻塞当前线程，直到该结果被提供。

```
public Future<String> calculateAsync() throws InterruptedException {
    CompletableFuture<String> completableFuture = new CompletableFuture<>();

    Executors.newCachedThreadPool().submit(() -> {
        Thread.sleep(500);
        completableFuture.complete("Hello");
        return null;
    });

    return completableFuture;
}
```

### 相继执行多个任务
假设我们需要调用两个远程 API：`firstApiCall()` 和 `secondApiCall()`。第一个 API 的结果将是第二个 API 的输入。如果使用 Future 接口，就无法异步地将这两个操作结合起来：
```
class Demo {
 public static void main(String[] args) throws ExecutionException, InterruptedException {
   ExecutorService executor = Executors.newSingleThreadExecutor();
   Future<String> firstApiCallResult = executor.submit(
           () -> firstApiCall(someValue)
   );
   
   String stringResult = firstApiCallResult.get();
   Future<String> secondApiCallResult = executor.submit(
           () -> secondApiCall(stringResult)
   );

   doOtherThings();
 }
}
```
在上面的代码示例中，我们通过在 `ExecutorService` 上提交一个返回 Future 的任务来调用第一个 API。我们需要将该值传递给第二个 API，但获取该值的唯一方法是使用我们之前讨论过的 Future 方法的 `get()`，而使用该方法会**阻塞主线程**。现在我们必须等到第一个 API 返回结果后再做其他事情。

```
class Demo {
  public static void main(String[] args) {

    var finalResult = CompletableFuture.supplyAsync(
         () -> firstApiCall(someValue)
    )
    .thenApply(firstApiResult -> secondApiCall(firstApiResult));

	doOtherThings();
  }
}
```

### 异步编排 CompletableFuture 的使用

#### 异步任务

`CompletableFuture` 提供了 `runAsync` 和 `supplyAsync` 来创建异步任务：

- 无返回值的异步任务：

```java
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    // 业务操作
    System.out.println("Task running...");	// 无返回值
});
```

- 有返回值的异步任务：

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // 业务操作
    return "Hello world";	// 有返回值
});
```

- 还可以指定线程池：

```java
ExecutorService executor = Executors.newFixedThreadPool(5);
CompletableFuture<String> Future  = CompletableFuture.supplyAsync(() -> {
    return "Hello world";
}, executor);
```

#### 支持链式调用，可以将多个异步任务串联起来

- 处理结果并返回新值：

  ```java
  CompletableFuture<String> result = future.thenApply(data -> data + " updated");
  ```

- 处理结果但不返回新值：

  ```java
  CompletableFuture<Void> future2 = future.thenAccept(data -> System.out.println("Result: " + data));
  ```

- 任务完成后执行操作：

  ```java
  future.thenRun(() -> System.out.println("Task finished"));
  ```

#### 并发任务

`CompletableFuture` 提供了 `allOf()` 和 `anyOf()` 方法来处理并发任务

- 所有任务完成后继续：

  ```java
  CompletableFuture<Void> all = CompletableFuture.allOf(
  	task1, task2, task3
  );
  ```

- 任意任务完成后继续：

  ```java
  CompletableFuture<Void> any = CompletableFuture.anyOf(
  	task1, task2, task3
  );
  ```

等待所有任务完成：
```java
all.join();
```

#### Spring 中 ThreadPool 的配置

```java
@Configuration
public class ThreadPoolConfig {
    
    @Bean
    public ThreadPoolExecutor threadPoolExecutor() {
        // 当前系统可用的处理器数量
        int processorsNum = Runtime.getRuntime().availableProcessors();
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
        		processorsNum * 2,					// corePoolSize
            	processorsNum * 2,					// maximunPoolSize
            	0,									// KeepAliveTime
            	TimeUnit.SECONDS,
            	new ArrayBlockingQueue<>(200),
            	Executors.defaultThreadFactory(),
            	// 自定义拒绝策略
            	(runnable, executor) -> {
                    try {
                        Thread.sleep(200);
                    } catch (Exception e) {
                        
                    }
                    // 再次将拒绝任务提交给线程池执行
                    executor.submit(runnable);
                }
        );
        // 线程池创建时，核心线程同时创建
        threadPoolExecutor.prestartAllCoreThreads();
        return threadPoolExecutor;
    }
}
```




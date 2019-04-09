# 线程池

## Runnable与Callable

## Executor
Exector在客户端与任务执行之间提供了一个中间层，可以替代Thread的工作，简化并发编程

```Java
public interface Executor {
  void execute(Runnable command);
}

public interface ExecutorService extends Executor {
  
  void shutdown();
	
  List<Runnable> shutdownNow();
	
  boolean isShutdown();
	
  boolean isTerminated();
	
  boolean awaitTermination(long timeout, TimeUnit unit)
			throws InterruptedException;
	
  <T> Future<T> submit(Callable<T> task);
	
  <T> Future<T> submit(Runnable task, T result);
	
  Future<?> submit(Runnable task);

  ....
}
```

通过观察`Executor`与`ExecutorService`接口，可以知道`executor`驱动`Runnable`对象，`submit`既可以驱动`Runnable`对象也可以驱动`Callable`对象

### SingleThreadExecutor
SingleThreadExecutor可以保证任意时刻在任何线程中都只有唯一的任务在运行，序列化执行任务可以消除资源同步的问题。



### 参考
- 《JAVA编程思想》
- 《Effective Java》

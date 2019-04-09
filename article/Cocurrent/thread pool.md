# 任务与执行机制
开发者尽量不要自己编写工作队列，也应该尽量不直接使用线程（`Thread`既可以充当工作单元，又可以是执行机制）。Executor Framework设计的理念是将工作单元（任务）与执行机制分开：工作单元接口为`Runnable`与`Callable`，执行任务的通用机制是`ExecutorService`。

## Thread

- 休眠

`TimeUnit.MIllISECONDS.sleep()`
- 优先级

`Thread.MAX_PRIORITY Thread.MIN_PRIORITY`
- 让步

`Thread.yield()`

- 后台线程（daemon）

后台线程是指在程序运行时候，在后台提供一种通用服务的线程，并且这种线程并不属于程序中不可或缺的部分。因此，当所有的非后台进程结束时，程序会终止，后台线程也会终止。`main()`是一个非daemon线程，使用方法为`thread.setDaemon(true)`并且要在线程启动之前设置此属性。

## ThreadFactory
编写定制的ThreadFactory可以定制由Executor创建的线程属性，Executors中每个静态的ExecutorService创建方法都被重载为接收一个ThreadFactory对象，而这个对象将被用来创建新的线程。

```Java
public interface ThreadFactory {
	Thread newThread(Runnable r);
}
```
`java.util.concurrent.Executors.DefaultThreadFactory`
```Java
static class DefaultThreadFactory implements ThreadFactory {
	private static final AtomicInteger poolNumber = new AtomicInteger(1);
	private final ThreadGroup group;
	private final AtomicInteger threadNumber = new AtomicInteger(1);
	private final String namePrefix;

	DefaultThreadFactory() {
	    SecurityManager s = System.getSecurityManager();
	    group = (s != null) ? s.getThreadGroup() :
				  Thread.currentThread().getThreadGroup();
	    namePrefix = "pool-" +
			  poolNumber.getAndIncrement() +
			 "-thread-";
	}

	public Thread newThread(Runnable r) {
	    Thread t = new Thread(group, r,
				  namePrefix + threadNumber.getAndIncrement(),
				  0);
	    if (t.isDaemon())
		t.setDaemon(false);
	    if (t.getPriority() != Thread.NORM_PRIORITY)
		t.setPriority(Thread.NORM_PRIORITY);
	    return t;
	}
}
```
## Executor
Exector在客户端与任务执行之间提供了一个中间层，可以管理Thread的对象，简化并发编程

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
```SingleThreadExecutor```可以保证任意时刻在任何线程中都只有唯一的任务在运行，如果想SingleThreadExecutor提交多个任务，那么任务会被放入悬挂任务队列。序列化执行任务可以消除资源同步的问题。因为这个执行器不是一个池（pool），所以名字不是SingleThreadPool。

### CachedThreadPool
```CachedThreadPool```在程序执行过程中通常创建与所需数量相同的线程，然后在它回收旧线程时停止创建新线程。

### FixedThreadPool
```FixedThreadPool```数量为1会变成SingleThreadExecutor。

### ScheduledThreadPool
```ScheduledThreadPool```用来替代`java.util.Timer`，数量为1会变成SingleThreadScheduledExecutor。

```Java
TimerTask task = new TimerTask() {
	@Override
	public void run() {
	}
};
Timer timer = new Timer();
long delay = 0;
timer.schedule(task, delay, period);
```

```Java
Executors.newSingleThreadScheduledExecutor().scheduleAtFixedRate(() -> {}, delay, period, TimeUnit.MINUTES);
```

## ThreadPoolExecutor

Executors 返回的线程池对象的弊端如下：
- `FixedThreadPool` 和 `SingleThreadExecutor`:
允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM。
- `CachedThreadPool` 和 `ScheduledThreadPool`:
允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM。

`ThreadPoolExecutor`是上述线程池（不包括`ScheduledThreadPool`）的父类，其构造方法有如下几种：
```Java
ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue)
 
ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, RejectedExecutionHandler handler) 

ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory) 
 
ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler) 
```



### 参考
- 《JAVA编程思想》
- 《Effective Java》
- 《阿里巴巴Java开发手册》

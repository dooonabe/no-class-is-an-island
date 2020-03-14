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

## Runnable与Callable
两者的区别: `Runnable`没有数据返回，`Callable`有数据返回，但是`ExecutorService`弥补了此项差异。通常任务返回结果的操作需要与`Future`和`CountDownLatch`配合使用。

`java.lang.Runnable`
```Java
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}
```

`java.util.concurrent.Callable`
```Java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```



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
	
	@Override
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
参数说明：
- corePoolSize 线程池的基本线程数
- maximumPoolSize 线程池中允许的最大线程数
- keepAliveTime 如果一个线程处在空闲状态的时间超过了该属性值，就会因为超时而退出，但是不会低于corePoolSize
- unit 时间单位
- workQueue 工作队列
- threadFactory 任务工厂
- handler 饱和策略：如果队列已满，并且当前线程数目也已经达到上限，那么意味着线程池的处理能力已经达到了极限，此时需要拒绝新增加的任务。至于如何拒绝处理新增的任务，取决于线程池的饱和策略RejectedExecutionHandler

ThreadPoolExecutor execute方法的实现：
```Java
public void execute(Runnable command) {
	if (command == null)
	    throw new NullPointerException();
	/*
	 * Proceed in 3 steps:
	 *
	 * 1. If fewer than corePoolSize threads are running, try to
	 * start a new thread with the given command as its first
	 * task.  The call to addWorker atomically checks runState and
	 * workerCount, and so prevents false alarms that would add
	 * threads when it shouldn't, by returning false.
	 *
	 * 2. If a task can be successfully queued, then we still need
	 * to double-check whether we should have added a thread
	 * (because existing ones died since last checking) or that
	 * the pool shut down since entry into this method. So we
	 * recheck state and if necessary roll back the enqueuing if
	 * stopped, or start a new thread if there are none.
	 *
	 * 3. If we cannot queue task, then we try to add a new
	 * thread.  If it fails, we know we are shut down or saturated
	 * and so reject the task.
	 */
	int c = ctl.get();
	// 如果正在执行的线程数小于corePoolSize，那么将此任务添加为线程，返回
	if (workerCountOf(c) < corePoolSize) {
	    if (addWorker(command, true))
		return;
	    c = ctl.get();
	}
	// 尝试将任务添加到workQueue
	if (isRunning(c) && workQueue.offer(command)) {
	    int recheck = ctl.get();
	    if (! isRunning(recheck) && remove(command))
		reject(command);
	    else if (workerCountOf(recheck) == 0)
                // 防止了SHUTDOWN状态下没有活动线程了，但是队列里还有任务没执行这种特殊情况。
                // 添加一个null任务是因为SHUTDOWN状态下，线程池不再接受新任务
		addWorker(null, false);
	}
	//如果添加到workQueue，说明线程池已经shutdown或者线程池已经达到饱和状态，所以reject这个任务
	else if (!addWorker(command, false))
	    reject(command);
}
```

创建自己的线程池：
```Java
//创建线程或线程池时请指定有意义的线程名称，方便出错时回溯
ThreadFactory factory = new ThreadFactoryBuilder().setDaemon(true).setNameFormat("consumer-queue-%d").setPriority(Thread.MAX_PRIORITY).build();
ThreadPoolExecutor executor = new ThreadPoolExecutor(5,10,60,TimeUnit.MINUTES,new ArrayBlockingQueue<Runnable>(100),factory);
executor.execute();
executor.shutdown();
```

## ThreadPoolExecutor与Thread相比的优势
1. 任务定义与任务执行机制解耦
2. 重用线程，减少线程创建与销毁的***巨大***开销

## CountDownLatch
循环不停的赛马，需要一圈赛完之后赛马在起跑线再次一同出发。

```Java
for(;;){
  // n 数量
  CountDownLatch latch = new CountDownLatch(n);
  for(int i = 0; i < n; i++)
    Future<Integer> result = executor.submit(()->{latch.countDown();return i;});  
}
```

# Thread Pool

## Thread Starvation DeadLock
在线程池中如果任务依赖其他任务，可能会产生死锁。

只有当任务都是*同类型的*并且*相互独立*时，线程池的性能才能达到最佳。

## 实现资源最优利用率
### CPU（计算）密集型任务
线程池大小=cpu_core_size + 1

### 包括I/O或阻塞操作任务
线程池大小=cpu_core_size*target_cpu_utilization*(1+w/c)

w/c:ratio of wait time to compute time
## 饱和策略
1. `AbortPolicy`
2. `DiscardPolicy`
3. `DiscardOldestPolicy`
4. `CallerRunsPolicy`
```Java

/**
* A handler for rejected tasks that runs the rejected task
* directly in the calling thread of the {@code execute} method,
* unless the executor has been shut down, in which case the task
* is discarded.
*/
public static class CallerRunsPolicy implements RejectedExecutionHandler {
/**
 * Creates a {@code CallerRunsPolicy}.
 */
public CallerRunsPolicy() { }

/**
 * Executes task r in the caller's thread, unless the executor
 * has been shut down, in which case the task is discarded.
 *
 * @param r the runnable task requested to be executed
 * @param e the executor attempting to execute this task
 */
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    if (!e.isShutdown()) {
	r.run();
    }
}
}
```

### 参考
- 《JAVA编程思想》
- 《Effective Java》
- 《阿里巴巴Java开发手册》

# BlockingQueue

条件等待的三元关系：加锁（持有条件队列相关的锁），条件谓词，条件变量。管理状态依赖性的机制必须与确保状态一致性的机制关联起来。
对于阻塞(有界)队列来说，条件谓词有两个：队列为空不允许take，队列已满不允许put。

## synchronized(内置条件队列与内置锁) unfair

```Java
public class DemoBlockingQueue {

    final Object[] queues;

    private int count = 0;

    public DemoBlockingQueue(int size) {
        this.queues = new Object[size];
    }

    public synchronized void put(Object o) throws InterruptedException {
        while (count == queues.length) {
            wait();
        }
        System.out.println(this.getClass() + ".add()");
        queues[count] = o;
        count++;
        notifyAll();
    }


    public synchronized Object take() throws InterruptedException {
        while (count == 0) {
            wait();
        }
        System.out.println(this.getClass() + ".offer()");
        Object o = queues[count-1];
        count--;
        notifyAll();
        return o;
    }
}
```


```Java
public class Test {
    public static void main(String[] args) throws InterruptedException {
        DemoBlockingQueue queue = new DemoBlockingQueue(1);
        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    System.out.println("thread1 start");
                    System.out.println("thread1:offer " + queue.take().toString());
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });
        thread1.start();

        Thread.sleep(10000);

        Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    System.out.println("thread2 start");
                    queue.put("element");
                    System.out.println("thread2:put element");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });
        thread2.start();
    }
}
/* result:
thread1 start
thread2 start
class com.dooonabe.book.DemoBlockingQueue.put()
thread2:put element
class com.dooonabe.book.DemoBlockingQueue.take()
thread1:take element
/*
```

## Lock unfair/fair

`java.util.concurrent.locks.Condition`中的**await**,**signal**,**signalAll**与`java.lang.Object`中的**wait**,**notify**,**notifyAll**相对应，在使用`Condition`时要使用正确的方法。

相较于上面的内置条件队列，使用`Condition`可以拆分条件谓词对应的队列，唤醒线程时可以避免不必要的线程上下文切换的消耗。

`java.util.concurrent.ArrayBlockingQueu`的设计思路就是一个锁两个队列。下面为简版的实现过程。

```Java
public class DemoBlockingQueueWithLock {
    // 共享对象可以使用final时，就是用final
    // 使用ReentranLock创建公平锁与非公平锁
    private final Lock lock = new ReentrantLock(true);
    // 分别为每个条件谓词创建条件队列
    private final Condition isFull = lock.newCondition();
    private final Condition isEmpty = lock.newCondition();

    private final Object[] queues;

    private int count = 0;

    public DemoBlockingQueueWithLock(int size) {
        this.queues = new Object[size];
    }

    public void put(Object o) throws InterruptedException {
        lock.lock();
        try {
            while (count == queues.length) {
                isFull.await();
            }
            System.out.println(this.getClass() + ".add()");
            queues[count] = o;
            count++;
            isEmpty.signal();
        } finally {
            lock.unlock();
        }
    }


    public Object take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) {
                isEmpty.wait();
            }
            System.out.println(this.getClass() + ".offer()");
            Object o = queues[count - 1];
            count--;
            // 调用signal或者Object的notify方法后，一定要尽快让线程结束锁的持有状态
            isFull.signal();
            return o;
        } finally {
            lock.unlock();
        }
    }
}
```


`java.util.concurrent.locks`
```Java
public interface Condition {

    /**
     * Causes the current thread to wait until it is signalled or
     * {@linkplain Thread#interrupt interrupted}.
     *
     * @throws InterruptedException if the current thread is interrupted
     *         (and interruption of thread suspension is supported)
     */
    void await() throws InterruptedException;

    /**
     * Causes the current thread to wait until it is signalled.
     *
     * <p>The current thread is assumed to hold the lock associated with this
     * {@code Condition} when this method is called.
     * It is up to the implementation to determine if this is
     * the case and if not, how to respond. Typically, an exception will be
     * thrown (such as {@link IllegalMonitorStateException}) and the
     * implementation must document that fact.
     */
    void awaitUninterruptibly();

    /**
     * Causes the current thread to wait until it is signalled or interrupted,
     * or the specified waiting time elapses.
     *
     * @param nanosTimeout the maximum time to wait, in nanoseconds
     * @return an estimate of the {@code nanosTimeout} value minus
     *         the time spent waiting upon return from this method.
     *         A positive value may be used as the argument to a
     *         subsequent call to this method to finish waiting out
     *         the desired time.  A value less than or equal to zero
     *         indicates that no time remains.
     * @throws InterruptedException if the current thread is interrupted
     *         (and interruption of thread suspension is supported)
     */
    long awaitNanos(long nanosTimeout) throws InterruptedException;

    /**
     * Causes the current thread to wait until it is signalled or interrupted,
     * or the specified waiting time elapses. This method is behaviorally
     * equivalent to:
     *  <pre> {@code awaitNanos(unit.toNanos(time)) > 0}</pre>
     *
     * @param time the maximum time to wait
     * @param unit the time unit of the {@code time} argument
     * @return {@code false} if the waiting time detectably elapsed
     *         before return from the method, else {@code true}
     * @throws InterruptedException if the current thread is interrupted
     *         (and interruption of thread suspension is supported)
     */
    boolean await(long time, TimeUnit unit) throws InterruptedException;

    /**
     * Causes the current thread to wait until it is signalled or interrupted,
     * or the specified deadline elapses.
     *
     * <p>The return value indicates whether the deadline has elapsed,
     * which can be used as follows:
     *  <pre> {@code
     * boolean aMethod(Date deadline) {
     *   boolean stillWaiting = true;
     *   lock.lock();
     *   try {
     *     while (!conditionBeingWaitedFor()) {
     *       if (!stillWaiting)
     *         return false;
     *       stillWaiting = theCondition.awaitUntil(deadline);
     *     }
     *     // ...
     *   } finally {
     *     lock.unlock();
     *   }
     * }}</pre>
     *
     * @param deadline the absolute time to wait until
     * @return {@code false} if the deadline has elapsed upon return, else
     *         {@code true}
     * @throws InterruptedException if the current thread is interrupted
     *         (and interruption of thread suspension is supported)
     */
    boolean awaitUntil(Date deadline) throws InterruptedException;

    /**
     * Wakes up one waiting thread.
     */
    void signal();

    /**
     * Wakes up all waiting threads.
     */
    void signalAll();
}
```


## Semaphore unfair/fair
无论使用内置锁还是显示锁，都需要小心再小心。尽可能使用JDK提供的并发工具是一个好办法。

Semaphore就像一个门卫，只有门卫允许进入门，才能继续进行操作。上面提到的条件谓词也是进门的许可。所以可以使用Semaphore替代条件谓词。

```Java
public class DemoBlockingQueueWithSemaphore {
    private final Semaphore semaphore;

    private final Object[] queues;
    // 索引定义为volatile
    // 与上面加锁模式下不同，因为加锁就保证了可见性
    private volatile int count = 0;

    // 实现线程安全的take方法
    private final Semaphore takeSemaphore;


    public DemoBlockingQueueWithSemaphore(int size) {
        // 可以定义公平与非公平队列
        semaphore = new Semaphore(size);
        this.queues = new Object[size];
        takeSemaphore = new Semaphore(1);
    }

    public void put(Object o) throws InterruptedException {
        // 获取不到许可的线程，会在此处阻塞
        semaphore.acquire();
        queues[count] = o;
        count++;
    }


    public Object take() throws InterruptedException {
        takeSemaphore.acquire();
        try {
            while (count == 0) {
                // wait 10 seconds
                TimeUnit.SECONDS.sleep(10);
            }
            Object o = queues[count - 1];
            count--;
            semaphore.release();
            return o;
        } finally {
            takeSemaphore.release();
        }
    }
}
```
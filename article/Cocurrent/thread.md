# 线程

## 线程安全
当多个线程访问同一个对象时，如果不用考虑这些线程在运行时环境下的调度和交替执行，也不需要进行额外的同步，或者在调用方进行任何其他的协调操作，（每次）调用这个对象的行为都可以获得正确的结果，那这个对象是线程安全的。

*we know when we see*

*线程安全类中封装了必要的同步机制，因此客户端无需进一步采取同步措施*。

### 线程安全的实现
- 阻塞（互斥）同步:`synchronized`,`ReetrantLock` 
- 非阻塞同步:`CAS`
- 无同步:`ThreadLocal`

## 线程死锁

### 产生死锁的必要条件

- 互斥

某种资源一次只允许一个进程访问，即该资源一旦分配给某个进程，其他进程就不能再访问，直到该进程访问结束。
- 占有且等待

一个进程本身占有资源（一种或多种），同时还有资源未得到满足，正在等待其他进程释放该资源。
- 不可抢占

别人已经占有了某项资源，你不能因为自己也需要该资源，就去把别人的资源抢过来。

- 循环等待

存在一个进程链，使得每个进程都占有下一个进程所需的至少一种资源。

#### 两种常见的死锁
- 获取锁的顺序不一致

- 资源饥饿

使用`ThreadPoolExecutor`或者`Executors`创建的线程池，当且仅当`workQueue`被填满时，才会创建大于`corePoolSize`且小于`maxPoolSize`的新的线程。

那么如果当前线程池中所有的线程提交依赖任务到线程池，并且没有填满工作队列，那么线程池是不会创建新的线程来执行依赖任务的，那么线程池就有可能一直停在当前的状态，造成死锁。

## 线程间的协作

```java
java.lang.Object

// Causes the current thread to wait until another thread invokes the **{@link java.lang.Object#notify()}** method or the **{@link java.lang.Object#notifyAll()}** method **for this object**. In other words, this method behaves exactly as if it simply performs the call {@code wait(0)}.
public final native void wait(long timeout) throws InterruptedException;


// Wakes up a single thread that is waiting **on this object's monitor**. If any threads are waiting on this object, one of them is chosen to be awakened. The choice is arbitrary and occurs at the discretion of the implementation. A thread waits on an object's monitor by calling one of the {@code wait} methods.
public final native void notify();


// Wakes up all threads that are waiting **on this object's monitor**. A thread waits on an object's monitor by calling one of the {@code wait} methods.
public final native void notifyAll();
```


```Java
java.lang.Thread

// Causes the currently executing thread to sleep (temporarily cease execution) for the specified number of milliseconds, subject to the precision and accuracy of system timers and schedulers. **The thread does not lose ownership of any monitors(不会释放任何锁)**.
public static native void sleep(long millis) throws InterruptedException;


// 不释放锁
// A hint to the scheduler that the current thread is willing to yield its current use of a processor. The scheduler is free to ignore this hint. **And The thread does not lose ownership of any monitors(不会释放任何锁)**.
public static native void yield();


// let main thread waits for this thread to die.
public final void join() throws InterruptedException {
    join(0);
}


public final synchronized void join(long millis)
throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {
        while (isAlive()) {
            wait(0);
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```

```Java
public static void main(String[] args) throws Exception {
    Thread thread = new Thread(()->{
            long times = 10L;
            while (--times > 0){
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("inner thread die");
        });
    // 设置为后台线程: 主线程运行完毕，程序即会退出；设置为非后台线程：主线程运行完，并不会立即退出，而会等待非daemon线程运行完毕后程序退出 
    thread.setDaemon(true);
    thread.start();
    // 子线程join()表示：主线程需等待子线程运行完毕，才能继续向下执行
    thread.join();
    System.out.println("main thread die");    
}
```

```Java
// wait notify 实现有界缓存
public class DemoBlockingQueue {

    final Object[] queues;

    private int count = 0;

    public DemoBlockingQueue(int size) {
        this.queues = new Object[size];
    }


    public void put(Object o) throws InterruptedException {
        synchronized (this) {
            while (count == queues.length) {
                wait();
            }
            queues[count] = o;
            count++;
            notifyAll();
        }
    }


    public Object take() throws InterruptedException {
        synchronized (this) {
            while (count == 0) {
                wait();
            }
            Object o = queues[count-1];
            count--;
            notifyAll();
            return o;
        }
    }
}
```

## 线程开销

1. 创建、销毁
2. 上下文切换
3. 内存同步

`synchronized`,`volatile`使用了内存屏障。内存屏障可以刷新缓存，使缓存无效，刷新硬件的写缓存，以及停止执行管道以及禁止指令的重排序。

4. 阻塞

发生阻塞之后，JVM可以采用**自旋等待**或者**上下文切换**来处理被阻塞的线程。
```Java
// 排号(ticket)自旋锁的实现
// do{} while(!unsafe.compareAndSwap(var1,var2,var3,var4))也是一种自旋
// 非适应性自旋
// 公平锁(ReentrantLock.FairSync)

private final Queue taskQueue = new ArrayBlockingQueue();

private int volatile num;

public synchronized void ticket(){
    queue.poll();
}

```

## 减少锁竞争(开销)
1. 减少锁的持有时间

缩小同步块代码，将实际不需同步执行的代码移除同步块，提高伸缩性。

> 伸缩性：当增加计算资源时，程序的吞吐量(多少)或者处理能力(多块)是否能相应的增加。在并发程序中，对伸缩性的主要威胁就是独占方式的资源锁。

2. 降低锁的请求频率

锁分解：全局锁减小为对象锁，对象锁减小为方法锁，方法锁减小为代码块锁。

锁分段：使用一组锁实现对整个对象的同步。

避免热点：计数器很容易成为热点域。
```Java
// ConcurrentHashMap 记录每一组的数量，总的数量是组的和
private transient volatile CounterCell[] counterCells;

final long sumCount() {
    CounterCell[] as = counterCells; CounterCell a;
    long sum = baseCount;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}
```

同时现代的JVM会对锁进行消除优化：

```Java
//同步锁只能由一个线程获得
synchronized(new Object()){
    // do something
}
```
```Java
//JVM escape analysis找出不会发布到堆的本地对象引用，进行锁消除
public String getStr(){
    // Vector是同步容器
    List<String> result = new Vector<String>();
    result.add("U2");
    return result.toString();
}
```

3. 使用带有协调机制的独占锁

尝试使用并发容器，读写锁，不可变对象，原子变量，消除同步变量(ThreadLocal)等更优化的工具。


## 实战
如果当多个线程访问同一个可变的状态变量时没有使用合适的同步，那么程序就会出现错误。有三种方法可以修复这个问题
### 不在线程之间共享该状态变量
- 问题场景

JStorm任务中缓存的数据对象，会被定时清理。如果使用另外的线程定时操作缓存对象，会造成多线程访问同一个可变的状态变量的情况，一不小心就会产生线程不安全问题。
- 解决方案

使用同步机制：设置tick tuple，使得操作缓存对象的线程只有一个。

### 将状态变量修改为不可变的变量


### 在访问状态变量时使用同步
- 问题场景

商品库存增减场景
- 解决方案

对同一商品ID的库存访问进行多线程同步。
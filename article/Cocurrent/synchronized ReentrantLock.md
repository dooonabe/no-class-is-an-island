# synchronized与ReentrantLock
两者都是互斥同步(互斥是因，同步是果，互斥是方法，同步是目的)的实现。在基本用法上，`ReetrantLock`与`synchronized`很相似，他们都具备一样的重入特性，只是代码写法上有点区别，一个表现为***API层面***（lock()和unlock()方法配合try/finally语句块完成）的互斥锁，另一个表现为***原生语法层面***的互斥锁。

## synchronized
方法级别的同步与方法内部代码块的同步都是使用Monitor实现的。

1. 方法级别
```Java
public synchronized void start();
descriptor: ()V
flags: ACC_PUBLIC, ACC_SYNCHRONIZED
Code:
    stack=2, locals=2, args_size=1
        0: ldc2_w        #2                  // long 1000l
        3: invokestatic  #4                  // Method java/lang/Long.valueOf:(J)Ljava/lang/Long;
        6: astore_1
        7: return
    LineNumberTable:
    line 16: 0
    line 17: 7
```
JVM根据方法区方法表结构中的ACC_SYNCHRONIZED访问标志判断方法是否为同步方法，在执行方法时检查方法的ACC_SYNCHRONIZED标志位是否被设置。

2. 方法内部代码块
```Java
public void run();
descriptor: ()V
flags: ACC_PUBLIC
Code:
    stack=2, locals=4, args_size=1
    0: aload_0
    1: dup
    2: astore_1
    3: monitorenter
    4: ldc2_w        #2      // long 1000l
    7: invokestatic  #4      // Method java/lang/Long.valueOf:(J)Ljava/lang/Long;
    10: astore_2
    11: aload_1
    12: monitorexit
    13: goto          21
    16: astore_3
    17: aload_1
    18: monitorexit
    19: aload_3
    20: athrow
    21: return
    Exception table:
        from    to  target type
            4    13    16   any
        16    19    16   any
    LineNumberTable:
    line 10: 0
    line 11: 4
    line 12: 11
    line 13: 21
    StackMapTable: number_of_entries = 2
    frame_type = 255 /* full_frame */
        offset_delta = 16
        locals = [ class com/dooonabe/SynchronizedMain, class java/lang/Object ]
        stack = [ class java/lang/Throwable ]
    frame_type = 250 /* chop */
        offset_delta = 4

```
monitorenter，monitorexit指令支持synchronized关键字。

## ReentrantLock
ReetrantLock(重入锁)的名字表明了其支持线程重入机制，也就是支持同步块对于同一条线程来说是可重入的。在这方面synchronized也是支持的。

`java.util.concurrent.locks.ReentrantLock`
```Java
// 重入机制的实现
abstract static Sync extends AbstractQueuedSynchronizer{
    ...
    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        // state == 1 表示资源被加锁，此时判断占有资源的线程是否为当前线程来实现重入机制
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
    ...
}

```

ReetrantLock底层使用了AQS框架，ReetrantLock与synchronized的不同之处有三个方面：

`java.util.concurrent.locks.Lock`
```Java
public interface Lock {

    void lock();

    void lockInterruptibly() throws InterruptedException;

    boolean tryLock();

    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

    void unlock();

    Condition newCondition();
}
```

- 等待可中断，可重试

`lockInterruptibly`,`tryLock`

sychronized是无条件的锁获取模式，只要有不一致的锁顺序就会发生**死锁**，一旦产生死锁就只能重启程序了。
```Java
public void transferMoney(){
    sychronized(A){
        sychronized(B){
            // do some transfer
        }
    }
}
```

ReentrantLock使用可定时或可轮询的锁获取模式，避免**死锁**的产生。
```Java
public void transferMoney(){
    while(true){
        if(A.lock.tryLock()){
            try{
                if(B.lock.tryLock()){
                    try{
                        // do some transfer
                    } finally{
                        B.locl.unlock();
                    }
                }
            } finally{
                A.lock.unlock();
            }
        }
    }
}
```

- 公平锁
```Java
/**
    * Sync object for fair locks
    */
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
        acquire(1);
    }

    /**
        * Fair version of tryAcquire. Don't grant access unless
        * recursive call or no waiters or is first.
        */
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        } // 支持重入
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```
- 非块结构加锁

`synchronized`要求锁的释放只能在与获得锁所在的虚拟机栈栈帧相同的栈帧中进行，而`Lock`可以跨方法(non-block-strunctured)获得锁与释放锁

- 锁绑定多个条件

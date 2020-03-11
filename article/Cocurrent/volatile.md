# volatile

## Java内存模型
1.原子性：基本数据类型读写，锁(synchronized)

2.可见性：final,volatile,锁(synchronized)

3.有序性：volatile,锁(synchronized)


## volatile是Java内存模型中的特殊规则。
1.可见性(保证变量对所有线程可见)

2.有序性(禁止指令重排序)

## volatile适合的场景
当且仅当满足下面三个条件时，才可以使用volatile：
1. 对变量的写入操作不依赖变量当前的值（或者开发者可以确认每个时刻只有一个线程在更新变量的值（让开发者保证是不可靠的））

2. 该变量不会与其他状态变量一起纳入不变性条件中

3. 在访问该变量是不需要加锁

```Java
volatile boolean asleep = false;

public void trySleep() {
    while (!asleep) {
        // do something
    }
    // asleep......
}
```
## volatile不适合的场景

### 控制多线程打印次数，错误版本
```Java
final static ExecutorService executorService = Executors.newFixedThreadPool(10);
static volatile int count = 0;

public static void main(String[] args) {
    // 闭锁是一种同步工具，可以延迟线程的进直到其到达终止状态
    final CountDownLatch start = new CountDownLatch(1);
    Runnable task = () -> {
        try {
            start.await();
            while (count < 10) {
                System.out.println(Thread.currentThread().getName() + ":" + count + "");
                count++;
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    };
    // 提交线程池之后，线程处于阻塞状态
    for (int i = 0; i < 10; i++)
        executorService.submit(task);
    // 线程阻塞状态取消
    start.countDown();
}
```
```Java
result:
pool-1-thread-2:0
pool-1-thread-1:0
pool-1-thread-2:1
pool-1-thread-1:2
pool-1-thread-2:3
pool-1-thread-1:4
pool-1-thread-1:6
pool-1-thread-1:7
pool-1-thread-2:5
pool-1-thread-2:9
pool-1-thread-1:8
```
上述程序并不是*we know it when we see it*，程序输出结果中出现了两个0（程序的输出结果是不定的，下一次可能会不同）。程序开发者使用volatile是想使用其可见性，控制多线程的输出结果，但是却违反了使用volatile规则的第一条：***对变量的写入操作不依赖变量当前的值***，`count++`违反了此规则。

## volatile与锁
加锁机制既可以保证可见性又可以保证原子性，volatile只能保证可见性。如果在访问变量的时候需要加锁，那么就不需用volatile修饰变量。

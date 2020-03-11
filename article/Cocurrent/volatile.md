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

## volatile与锁
加锁机制既可以保证可见性又可以保证原子性，volatile只能保证可见性。如果在访问变量的时候需要加锁，那么就不需用volatile修饰变量。

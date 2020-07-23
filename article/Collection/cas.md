# CAS(Compare-and-Swap)
CAS指令需要有三个操作数，分别是内存地址V，旧的预期值A，新值B。CAS指令执行时，当且仅当V符合旧预期值A时，处理器用新址B更新V的值，否则就不执行更新，但无论是否更新了V的值，都会返回V的旧值，上述的处理过程是一个原子操作。

## ABA

使用**版本号+AtomicReference**解决ABA问题。

## AtomicInteger实现原理
1. volatile: 保证变量内存性
2. CAS: 保证操作原子性

`java.util.concurrent.atomic.AtomicInteger`

```Java
static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}

private volatile int value;

public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}

sun.misc.Unsafe
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```
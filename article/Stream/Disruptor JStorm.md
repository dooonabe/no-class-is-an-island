# Disruptor在JStorm中的应用

无法充分使用缓存行特性的现象，称为伪共享(false sharing)。

## 缓存行
```Java
public class ArrayBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, Serializable{

...

    /** The queued items */
    final Object[] items;

    /** items index for next take, poll, peek or remove */
    int takeIndex;

    /** items index for next put, offer, or add */
    int putIndex;

    /** Number of elements in the queue */
    int count;
...
}
```

缓存行(cache line)一般64byte，一个int占用4byte，所以`ArrayBlockingQueue`中面向生产者的指针`putIndex`与面向消费者的`takeIndex`有很大可能被放到同一个缓存行中，这样会产生伪共享问题，即生产者更新`putIndex`时，`takeIndex`会被重新从主内存中加载到缓存行。


1. `手动填充缓存行`

2. `@sun.misc.Contended`

## Disruptor compare with ArrayBlockingQueue

1. cas
```Java
public long tryNext(int n) throws InsufficientCapacityException
{
    if (n < 1)
    {
        throw new IllegalArgumentException("n must be > 0");
    }
 
    long current;
    long next;
 
    do
    {
        current = cursor.get();
        next = current + n;
 
        if (!hasAvailableCapacity(gatingSequences, n, current))
        {
            throw InsufficientCapacityException.INSTANCE;
        }
    }
    while (!cursor.compareAndSet(current, next));
 
    return next;
}
```
2. 
3.

## how does JStorm to use Disruptor




## 参考
- [高性能队列——Disruptor](https://tech.meituan.com/2016/11/18/disruptor.html)
- [伪缓存](http://ifeve.com/falsesharing/)

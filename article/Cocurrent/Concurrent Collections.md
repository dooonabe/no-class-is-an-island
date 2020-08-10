# 并发容器

## Future

## Semaphore

## CountDownLatch

线程同一时刻开始计算
- volatile
缺点：线程没计算之前，不会释放cpu资源

- wait/notifyAll + thread.setDaemon(false)

- CountDownLatch

## BlockingQueue

## ReentrantReadWriteLock
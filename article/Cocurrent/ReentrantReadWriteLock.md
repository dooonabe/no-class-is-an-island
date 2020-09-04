# 读写锁

```Java
public static class ReadLock implements Lock, java.io.Serializable {
    private static final long serialVersionUID = -5992448646407690164L;
    private final Sync sync;

    /**
        * Constructor for use by subclasses
        *
        * @param lock the outer lock object
        * @throws NullPointerException if the lock is null
        */
    protected ReadLock(ReentrantReadWriteLock lock) {
        sync = lock.sync;
    }

    /**
        * Acquires the read lock.
        *
        * <p>Acquires the read lock if the write lock is not held by
        * another thread and returns immediately.
        *
        * <p>If the write lock is held by another thread then
        * the current thread becomes disabled for thread scheduling
        * purposes and lies dormant until the read lock has been acquired.
        */
    public void lock() {
        sync.acquireShared(1);
    }

    /**
        *
        * @throws InterruptedException if the current thread is interrupted
        */
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }

    /**
        *
        * @return {@code true} if the read lock was acquired
        */
    public boolean tryLock() {
        return sync.tryReadLock();
    }

    /**
        *
        * @param timeout the time to wait for the read lock
        * @param unit the time unit of the timeout argument
        * @return {@code true} if the read lock was acquired
        * @throws InterruptedException if the current thread is interrupted
        * @throws NullPointerException if the time unit is null
        */
    public boolean tryLock(long timeout, TimeUnit unit)
            throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
    }

    /**
        * Attempts to release this lock.
        *
        * <p>If the number of readers is now zero then the lock
        * is made available for write lock attempts.
        */
    public void unlock() {
        sync.releaseShared(1);
    }

    /**
        * Throws {@code UnsupportedOperationException} because
        * {@code ReadLocks} do not support conditions.
        *
        * @throws UnsupportedOperationException always
        */
    public Condition newCondition() {
        throw new UnsupportedOperationException();
    }

    /**
        * Returns a string identifying this lock, as well as its lock state.
        * The state, in brackets, includes the String {@code "Read locks ="}
        * followed by the number of held read locks.
        *
        * @return a string identifying this lock, as well as its lock state
        */
    public String toString() {
        int r = sync.getReadLockCount();
        return super.toString() +
            "[Read locks = " + r + "]";
    }
}
```
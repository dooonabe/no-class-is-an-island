# Exclusive Lock & Shared Lock

想象***黑板***是需要同步的对象，操作***黑板***的对象包括一位老师，很多位学生。当老师在黑板上写字时，就是排他锁的场景:
>Think of a lockable object as a blackboard (lockable) in a class room containing a teacher (writer) and many students (readers).
While a teacher is writing something (exclusive lock) on the board:
- 所有人都不能读***黑板***上的内容。因为老师获得了排他锁，其他的共享锁不能获得。
>- Nobody can read it, because it's still being written, and she's blocking your view => If an object is exclusively locked, shared locks cannot be obtained.
- 其他老师也不能在***黑板***上书写，因为这样会让学生们困惑。如果一个对象上有排他锁，那么其他排他锁就不能再操作这个对象。
>- Other teachers won't come up and start writing either, or the board becomes unreadable, and confuses students => If an object is exclusively locked, other exclusive locks cannot be obtained.

当学生们读***黑板***上的内容时，就是共享锁的场景:
>When the students are reading (shared locks) what is on the board:

- 学生们可以一起读***黑板***上的内容。不同共享锁可以同时操作同一个对象。
>- They all can read what is on it, together => Multiple shared locks can co-exist.
- 老师要等着学生们读完***黑板***上的内容之后才能再写新的内容。如果对象上有共享锁，那么排他锁需要等待。
>- The teacher waits for them to finish reading before she clears the board to write more => If one or more shared locks already exist, exclusive locks cannot be obtained.

# ZooKeeper
## exclusive lock
## shared lock

# Redis
## exclusive lock
## shared lock

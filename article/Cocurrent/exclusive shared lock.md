# Exclusive Lock & Shared Lock

Think of a lockable object as a blackboard (lockable) in a class room containing a teacher (writer) and many students (readers).
While a teacher is writing something (exclusive lock) on the board:
- Nobody can read it, because it's still being written, and she's blocking your view => If an object is exclusively locked, shared locks cannot be obtained.

- Other teachers won't come up and start writing either, or the board becomes unreadable, and confuses students => If an object is exclusively locked, other exclusive locks cannot be obtained.

When the students are reading (shared locks) what is on the board:

- They all can read what is on it, together => Multiple shared locks can co-exist.

- The teacher waits for them to finish reading before she clears the board to write more => If one or more shared locks already exist, exclusive locks cannot be obtained.

# ZooKeeper

# Redis

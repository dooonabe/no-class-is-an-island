# 快速报错（fail-fast）机制

Java容器有一种保护机制，它会探查容器上的任何除了你的进程所进行的操作以外的所有变化，一旦它发现其他进程修改了容器，就会立刻抛出`ConcurrentModificationException`异常。这就是“快速报错”的意思——不是使用复杂的算法在事后来检查问题。

### 原理



`ConcurrentHashMap` `CopyOnWriteArrayList` `CopyOnWriteArraySet` 都使用了可以避免`ConcurrentModificationException`的技术

# synchronized与ReentrantLock
两者都是互斥同步(互斥是因，同步是果，互斥是方法，同步是目的)的实现。在基本用法上，ReetrantLock与synchronized很相似，他们都具备一样的重入特性，只是代码写法上有点区别，一个表现为***API层面***（lock()和unlock()方法配合try/finally语句块完成）的互斥锁，另一个表现为***原生语法层面***的互斥锁。

## synchronized


## ReentrantLock
ReetrantLock(重入锁)的名字表明了其支持线程重入机制，也就是支持同步块对于同一条线程来说是可重入的。在这方面synchronized也是支持的。ReetrantLock与synchronized的不同之处有三个方面：

- 


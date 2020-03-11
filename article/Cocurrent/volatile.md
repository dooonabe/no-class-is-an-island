# volatile

## Java内存模型
1.原子性：基本数据类型读写，锁(synchronized)
2.可见性：final,volatile,锁(synchronized)
3.有序性：volatile,锁(synchronized)


## volatile是Java内存模型中的特殊规则。
1.可见性(保证变量对所有线程可见)
2.有序性(禁止指令重排序)

## case


## volatile与锁
加锁机制既可以保证可见性又可以保证原子性，volatile只能保证可见性。如果在访问变量的时候需要加锁，那么就不需用volatile修饰变量。



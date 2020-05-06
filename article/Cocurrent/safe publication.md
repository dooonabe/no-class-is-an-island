# 安全发布
一切都是`Java Memory Model`（`JMM`,Java内存模型）引发的问题，但是也不能怪`Java Memory Model`做的不够好，设计总是有两面性？


安全发布：安全发布的对象，保证**对象的引用**以及**对象的状态**必须同时对其他线程**可见**。

## JMM(Java内存模型)

1. 缓存一致性

允许不同的处理器在任意时刻从同一存储位置上看到不同的值。

2. 指令重排

### Happens-Before规则
 1. 程序顺序规则

 2. 监视器锁规则
 
 在监视器锁上的解锁操作必须在**同一个监视器锁**上的加锁操作之前执行。
```Java
pulic class UnsafeLazyInitialization{
    // todo 为什么不需要声明volatile???
    private static Resource resource;

    public synchronized static Resource getInstance(){
        if(resource == null)
            resource = new Resource();
        return resource;
    }
}
```

 3. volatile变量规则
 
 对volatile变量的写入操作必须在对该变量的读操作之前完成。
 
 4. 

 5.
 6.

  


## 安全发布的常用模式
1. 在**静态初始化函数**中初始化一个对象的引用
```Java
pulic class UnsafeLazyInitialization{
    // 静态初始化函数
    private static Resource resource = new Resource();

    public static Resource getInstance(){
        return resource;
    }
}
```
2. 将对象的引用保存到volatile类型的域或者AtomicReference对象中
3. 将对象的引用保存到某个正确构造对象的final类型域中
4. 将对象的引用保存到一个由锁保护的域中


## 延迟初始化

既可以降低程序启动速度，又可以避免没有实际作用的高开销操作。

```Java
pulic class UnsafeLazyInitialization{
    private static Resource resource;

    public static Resource getInstance(){
        if(resource == null)
            resource = new Resource();
        return resource;
    }
}
```

上述的延迟初始化(单例模式)会存在两个问题：

1. 竞态条件问题，会new多个`Resource`实例
2. 其他线程看到对部分构造的`Resource`实例的引用

线程间没有遵循`Happens-Before`规则，可能线程A写入resource的操作发生在线程B读取resource的操作之后，因此线程b不一定能看到Resource的正确状态。

> 除了不可变变量以外，使用被**另一个线程初始化的对象**通常都是不安全的，除非对象的发布操作在使用该对象的线程开始使用之前执行。

解决上述存在缺陷的延迟初始化的方法有很多，其中最简单的是用`synchronized`修饰`getInstance()`，此外还可以使用**占位类模式**。

### 占位类模式

```Java
pulic class SafeLazyInitialization{
    // 静态内部类会在第一次被访问的时候，被JVM初始化
    private static class ResourceHolder{
        public static Resource resource = new Resource();
    }

    public static Resource getInstance(){
        return ResourceHolder.resource;
    }
}
```
### DCL(双重检查加锁)存在的问题

```Java
pulic class UnsafeLazyInitialization{
    // 如果将resource修饰为volatile，那么就可以借助Happens-Before解决DCL存在的问题
    private static Resource resource;

    public static Resource getInstance(){
        if(resource == null){
            // Happens-Before原则在这里保证了，再次进入同步块的线程不会new出新的实例
            synchronized(UnsafeLazyInitialization.class){
                if(resource == null)
                    resource = new Resource();
            }
        }
        // 在Happens-Before之外，可能resource处于部分构造的状态
        return resource;
    }
}
```





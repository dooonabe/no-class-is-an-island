#高效操作Redis数据库

Redis数据库是一个支持分布式、基于内存存储的key-value数据库，同时提供数据持久化到磁盘的功能。Redis数据库支持多种多样的数据结构，其中包括字符串（strings）,列表（lists）, 哈希表（maps）, 集合（sets）,有序集合（sorted sets）等，Redis数据库也为这些数据结构提供了丰富的操作方法。

尽管Redis是一个高性能的数据库，但是只有开发人员正确合理的使用，才可以发挥出Redis的优势。

在使用Redis数据库的过程中，笔者总结了一些使用Redis数据库的注意事项。

1.避免短时间内频繁查询同一个key
问题展示：

在对程序性能优化的过程中，笔者在Redis集群的某节点下发现了如上图所示的操作日志
查看执行日志：在redis的bin目录下执行 sh ./redis-cli monitor可以查看当前节点的操作日志

问题说明：
短时间内（1毫秒以内）程序不停地获取Redis中的某个key是没有意义的。一方面这些get操作得到的数据是不变的，另一方面这些大量的不必要操作会白白浪费Redis资源。

问题解决：
请想一下，怎么样才能避免掉这些重复的操作呢。
开发人员在程序中使用本地缓存是一个不错的办法——程序查询Redis之前首先在本地缓存中做一次查询。

详细的思考一下，要如何设计这样的本地缓存呢，它要具有什么样的功能呢。

1.缓存要有最大值限制。如果不限制本地缓存的大小，程序会存在内存溢出的风险，并且体积过大的本地缓存也会影响查询效率
2.缓存要有失效机制。对于缓存来说失效时间很重要，超过一定的时间缓存不应当再被使用。同时本地缓存最好可以统计每个缓存的访问频率或者缓存的上一次操作时间，这样在本地缓存空间不足时，可以清除掉那些被访问次数少或者很久没有被访问的缓存项
3.缓存要是线程安全的。如果本地缓存是不安全的，那么多个线程访问同一个缓存时可能会取到不同的值。

看到这里，你是不是已经在思考如何实现具有这么多功能的缓存结构了呢。
造轮子可以提升开发者的思考力，但是项目时间紧的时候，选择最适合的轮子拿来用也不错。Google开源的Java库Guava是一个好轮子。Guava目前在Github上已经有接近28K个star，可见其在Java开发者中多么受欢迎。那么要如何使用好这个轮子呢。

二话不说，请看代码：

1.加载缓存
实现抽象类CacheLoader<k,v>的v load(k var1)方法，指定缓存要从哪里获取到。当开发者从cahce中取一个不存在的缓存项时，会出触发load操作，加载缓存到cahce
2.缓存回收

基于容量的回收
maximumSize规定了缓存项数目的最大值。在cache中的缓存项的条数达到限定值之前，缓存就可能进行回收操作。cache将尝试回收最近没有使用或总体上很少使用的缓存项。

定时回收
目前有两种定时回收的方法：
A.expireAfterAccess(long, TimeUnit)：缓存项在给定时间内没有被读/写访问，则回收。这种缓存的回收顺序和基于大小回收一样。
B.expireAfterWrite(long, TimeUnit)：缓存项在给定时间内没有被写访问（创建或覆盖），则回收。如果认为缓存数据总是在固定时候后变得陈旧不可用，这种回收方式是可取的。
cache不会主动清除超时的缓存项，需要开发者主动调用cleanup()方法

显示清除
任何时候都可以显式地清除缓存项：
A.个别清除：cache.invalidate(key)
B.批量清除：cache.invalidateAll(keys)
C.清除所有缓存项：cache.invalidateAll()
3.移除监听器

RemovalCause中包含缓存项被移除的原因：替换，超时，大小等，开发者可以对由于不同原因被移除的缓存项做相应的处理。

更多Guava Cache操作请参阅相关API手册

2.多用复合指令，减少与Redis交互次数
问题展示：

上图是笔者在程序部署节点使用ping命令查看与 Redis集群某个节点的连通情况，通过观察可以看到连通耗时在130纳秒左右。

上图是Redis集群某节点的执行耗时情况，图中类型的操作耗时在50纳秒左右

问题说明：
在生产环境中，访问Redis集群的程序往往与Redis集群不在同一个服务器节点上，这样就可能导致操作Redis的网络IO耗时比Redis执行命令的耗时更多。
如果可以减少网络IO的耗时，是不是就可以让Redis多执行一些命令呢。
问题解决：
Redis是一种基于客户端-服务端模型以及请求/响应协议的TCP服务。
通常情况下一个请求会遵循以下步骤：
A.客户端向服务端发送一个查询请求，并监听Socket返回，通常是以阻塞模式，等待服务端响应。
B.服务端处理命令，并将结果返回给客户端。
下面是4个命令序列执行情况：
    Client: INCR X
    Server: 1
    Client: INCR X
    Server: 2
    Client: INCR X
    Server: 3
    Client: INCR X
    Server: 4

聪明的你肯定想到了，要减少网络IO，一次传输好几个命令让Redis执行不就可以了。
命令序列执行就像这样：
    Client: INCR X
    Client: INCR X
    Client: INCR X
    Client: INCR X
    Server: 1
    Server: 2
    Server: 3
    Server: 4

那么在实际应用中要怎么操作呢。

1.批量执行命令
目前Java操作Redis主要有两个jar包可以选择，一个是Jedis库，一个是Redission库。以Jedis库为例说明，Jedis提供了很多批处理命令供开发者使用，像mget,mset这样的批量get/set方法；另外还有set数据到redis时，使用包含设置ttl的结合命令——set(final String key, final String value, final String nxxx, final String expx, final long time) 等等。

2.Pipeline
批量执行命令可以被看做简化版的pipeline实现，因为开发者可以将不同类型的操作都在放同一个pipeline中执行。

Jedis样例：
目前Jedis 最新版本2.9.0没有提供集群模式下使用pipeline的方法，单个节点使用pipeline时，首先创建一个pipeline，之后将要操作Redis的命令添加到pipeline中，最后执行pipeline获取返回结果集。

代码如下：

pipeline处理的key要分布在同一个节点上，因为jedis表示集群中的某一个节点

Redission样例：
Redission很好地支持了在Redis集群上使用pipeline。

代码如下：

3.及时清理过期数据
问题展示：

问题说明：
使用ttl 命令可以查看key的有效期，单位为秒，如果ttl返回值为-1，那么表示这个key如果不被用户主动删除，会永久存在。存放无用并且没有失效时间的数据，会造成内存资源地浪费。
问题解决：
1.程序及时删除掉Redis中的数据
2.程序放置数据到Redis时设置数据过期时间，之后Redis会自动清除过期数据



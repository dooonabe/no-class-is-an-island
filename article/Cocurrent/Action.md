# 实战

## 实现资源最优利用率
### CPU（计算）密集型任务
线程池大小=cpu_core_size + 1

### 包括I/O或阻塞操作任务
线程池大小=cpu_core_size*target_cpu_utilization*(1+w/c)

w/c:ratio of wait time to compute time

## 同步影响性能
### 现象
运维人员反应在每天九点半左右，数据接收服务消费能力直线下降。

### 观察数据
- jps -lv 查询进程的lvmid
- jstack lvmid 观察是否有处于BLOCKED状态的线程

```Java
    java.lang.Thread.State: BLOCKED (on object monitor)
    at org.apache.log4j.Category.callAppenders(Category.java:204)
    - waiting to lock <0x00000000e5915bd8> (a org.apache.log4j.spi.RootLogger)
    at org.apache.log4j.Category.forcedLog(Category.java:391)
    at org.apache.log4j.Category.log(Category.java:856)
```

### 查看代码
```Java
void callAppenders(LoggingEvent event) {
  int writes = 0;

  for(Category c = this; c != null; c=c.parent) {
    // Protected against simultaneous call to addAppender, removeAppender,...
    synchronized(c) {
  if(c.aai != null) {
    writes += c.aai.appendLoopOnAppenders(event);
  }
  if(!c.additive) {
    break;
  }
    }
  }

  if(writes == 0) {
    repository.emitNoAppenderWarning(this);
  }
}
```
### 结论
日志工程的代码中存在`synchronized`方法，导致很多调用日志对象的方法在高并发情况下出现了阻塞。升级log4j可以解决此问题。
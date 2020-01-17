# 实战

## 版本太低，影响性能
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


## 线程数量的合理值


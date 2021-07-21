# Spark
## RDD
操作类型
- transformation

transformation会根据当前RDD来定义新的RDD
- action

action是从RDD返回值

### 键值对RDD


## 共享变量

### 广播变量（broadcast variable）
### 累加变量（accumulator）

## Spark任务调度
- 用户使用spark-submit命令提交一个Spark应用程序
- spark-submit在集群上启动驱动进程，并调用由用户指定的main方法
- 驱动进程联系**集群管理器**，根据提供的配置参数来请求启动执行进程JVM所需的资源
- **集群管理器**在工作节点上启动执行进程JVM
- 驱动进程扫描用户应用程序。根据程序中的RDD动作与变化，Spark创建运算图
- 当调用一个动作时，图会被提交到一个DAG调度程序。DAG调度程序会将运算图划分为一些**阶段**
- 一个**阶段**由基于输入数据分区的任务组成。DAG调度程序会通过流水线把运算符连一起，从而优化运算图。DAG调度程序的最终结果是一组阶段
- 这些**阶段**会被传递到任务调度程序。任务调度程序通过**集群管理器**启动任务
- 任务在执行进程上运行，从而计算和保存结果
- 如果驱动程序的`main`方法退出，或者调用`SparkContext.stop()`,那么驱动程序就会终止执行进程并从集群管理器释放资源


## 参考
- [Spark性能优化指南——基础篇](https://tech.meituan.com/2016/04/29/spark-tuning-basic.html)
- [Spark性能优化指南——高级篇](https://tech.meituan.com/2016/05/12/spark-tuning-pro.html)

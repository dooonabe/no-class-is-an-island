# 2020.10 - 2021.07

## 考评系统
1. [scrum敏捷开发](https://www.scrumguides.org/docs/scrumguide/v2017/2017-Scrum-Guide-Chinese-Simplified.pdf)
```
- 透明 审查(与最终的目标是否有偏差) 适应
- sprint
计划会议：做什么，怎么做
每日站会：昨天工作，今天工作，遇到的问题
审核会议：本次周期完成的内容是否满足mp的要求
总结会议：“如果再来一次，我们能不能做得更好”

- 产品负责人-开发团队-scrum master
```
2. 使用Mac设备对ios移动设备进行web调试
3. [Mybatis源码阅读](https://my.oschina.net/wangzhenchao/blog/4307101)
4. Mybatis连接多数据源
5. Database View
6. git clone使用ssh协议
7. Git Commit说明规范化
```
feat：新功能（feature）
fix：修补bug
docs：文档（documentation）
style： 格式（不影响代码运行的变动）
refactor：重构（即不是新增功能，也不是修改bug的代码变动）
test：增加测试
chore：构建过程或辅助工具的变动
```
6. Git WorkFlow
7. Git 合并不相干branch
```
git remote add origin xxx.git
git pull--allow-unrelated-histories
git branch --set-upstream-to=origin/master master
```

## 数据可视化
1. DSL
2. 递归JSON对象/递归查找文件
3. Spark SQL 数据倾斜 [Spark性能优化指南——基础篇](https://tech.meituan.com/2016/04/29/spark-tuning-basic.html) [Spark性能优化指南——高级篇](https://tech.meituan.com/2016/05/12/spark-tuning-pro.html)
4. 统计任务幂等性设计
5. @ControllerAdvice @ExceptionHandler
6. Filter Interceptor
7. Maven: properties/dependencyManagement
8. docker kubernetes

## ETL
1. 如何动态加载jar
2. 内部类会被单独编译为A$B.class
3. 可视化任务编排与任务数据解析:DAG的遍历
4. CountDownLatch ThreadLocal
5. XXL RPC
6. 自定义反序列化实现：com.alibaba.fastjson.parser.deserializer.ObjectDeserializer
7. 查找并读取classpath下的文件：Files.readAllLines(Paths.get(this.getClass().getResource("res.txt").toURI()), Charset.defaultCharset())
8. flink

## Article
- [Mybatis与fastjson是如何处理特定类型的转化]()
- [使用HBase列族实现秒级数据血缘追溯的方案]()
- [Datax如何实现插件jar的动态加载，与JDBC有哪些相同](https://blog.csdn.net/soaring0121/article/details/102552157 https://www.jianshu.com/p/09f73af48a98)
- [Java HTTP客户端最佳实践]()

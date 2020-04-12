# 实战

## CPU占用高
### 思路
1. top -p pid: 查看目标进程信息
2. shift h: 查看进程中所有线程的资源使用情况
3. jstack pid | grep (printf '%x' pid(线程id)): 查看某线程对应jstack中的线程信息
4. 详细分析涉及代码块

## java.lang.RuntimeException: java.io.InvalidClassException: local class incompatible: stream classdesc serialVersionUID,local class serialVersionUID

### 原因描述
you create your serialized data with a given library A (version X), then you try to read this data with the same library A (but version Y)
### 思路
1. [《深入理解Java虚拟机》](https://book.douban.com/subject/24722612/)在介绍老版本tomcat的类加载实现中，介绍了tomcat如何兼容多spring版本——也就是隔离不通版本的相同类(相同类路径的类)
2. jps -lmv: 查看完整的java启动命令
3. 关注java的cp参数值，是否存在两个包中存在相同类的情况。


## GC对array对象的影响最小 


## 如何降低Spring Cloud工程的jar包大小以及去除无效依赖让heap变小
1. 类只有被用到才会被虚拟机初始化(加载到方法区)
2. 使用`mvn dependency:analyze`分析依赖情况，手动删去无效的依赖引用
3. 源码包与依赖包分离，启动源码包时用classpath指定依赖包目录
```Shell
java –classpath ... [same as (java -cp ...)]
java -Dloader.path= ...
```

## 参考
- [Java classpath](https://howtodoinjava.com/java/basics/java-classpath/)

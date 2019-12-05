# 实战
## CPU占用高
### 思路
1. top -p pid: 查看目标进程信息
2. shift h: 查看进程中所有线程的资源使用情况
3. jstack pid | grep (printf '%x' pid(线程id)): 查看某线程对应jstack中的线程信息
4. 详细分析涉及代码块

## java.lang.RuntimeException: java.io.InvalidClassException:local class incompatible: stream classdesc serialVersionUID = , local class serialVersionUID =




## 参考

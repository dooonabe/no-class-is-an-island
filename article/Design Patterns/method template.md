# 模板方法模式
*深入封装算法块，让子类可以在任何时候都可以将自己挂接到运算里*

### 定义
模板方法模式 在一个方法中定义一个算法骨架，而将一些步骤延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。

*模板方法定义了一个算法的步骤，并允许子类为一个或者多个步骤提供实现*

```Java
public abstract class MakeCard{
  
  void final start{
    step1();
    step2();
    step3();
    step4();
  }
  
  void step1(){
  // start
  } 
  
  abstract void  step2();
  
  abstract void  step3();
  
  void step4(){
  //end
  }
}
```
### 定义钩子，改变算法流程
```Java
public abstract class MakeCard{
  
  void final start{
    step1();
    step2();
    if(hook()){
    step3();
    }
    step4();    
  }
  
  void step1(){
  // start
  } 
  
  abstract void  step2();
  
  abstract void  step3();
  
  void step4(){
  //end
  }
  
  boolean hook(){
    return true;
  }
}
```
算法骨架中如果有可选择的部分，那么子类可以选择实现钩子方法，从而控制流程。

### 好莱坞原则
*别调用（打电话给）我们，我们会调用（打电话给）你*


避免高层与低层组件之间有明显的环状依赖

### 与策略模式的比较

策略模式的核心是**对象组合**与**委托模型**


模板方法模式是最常被使用的设计模式

### 优点
- 消除代码冗余
- 解耦业务逻辑与通用流程
- 开放包容，方便业务扩展

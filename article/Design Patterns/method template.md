# 模板方法模式
*深入封装算法块，让子类可以在任何时候都可以将自己挂接到运算里*

### 定义
模板方法模式 在一个方法中定义一个算法骨架，而将一些步骤延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤

*模板方法定义了一个算法的步骤，并允许子类为一个或者多个步骤提供实现*

```
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

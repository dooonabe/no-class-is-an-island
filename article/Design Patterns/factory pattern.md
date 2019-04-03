# 工厂模式

### 简单工厂
简单工厂其实不是一个设计模式，反而比较像一种编程习惯。
```Java
public class SimpleKuduFactory{
  public KuduTable createTable(Map config){
    // do something
    return kuduTable;
  }
}

public class Kudu{
  SimpleKuduFactory factory;
  
  public Kudu(SimpleKuduFactory factory){
    this.factory = factory;
  }

  public void start(){
    factory.createTable(config);
    // do something
  }
}

```

### 工厂方法（factory method pattern）
工厂方法用来处理对象的创建，并将这样的行为封装在**子类**中。这样客户程序中关于超类的代码就和子类对象的创建代码**解耦**(decouple)了。
```Java
public class Abstract Kudu{
  
  public void start(){
    createTable(config);
    // do something
  } 
  
  protected abstract KuduTable createTable(Map config);
  
}
```
所有的工厂模式都用来**封装对象的创建**。工厂方法模式通过让**子类**决定该创建的对象是什么，来达到将对象创建的过程封装的目的。

指导方针：
- 变量不可以持有具体类的引用
- 不能让类派生自具体类
- 不要覆盖基类中已实现的方法

### 抽象工厂



### 参考
- 《Head First设计模式》






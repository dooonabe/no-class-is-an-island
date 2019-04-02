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




### 参考
- 《Head First设计模式》






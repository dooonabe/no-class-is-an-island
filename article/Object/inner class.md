# Inner Class

## 非静态内部类
在拥有外部类对象之前不可能创建内部类对象。
```Java
OuterClass out = new OuterClass();
OuterClass.InnerClass = out.new InnerClass();
```


## 静态内部类（嵌套类）
静态内部类与外围内部类没有联系。

## 多继承
```Java
class D{}

abstract class E{}

class X extends D{
  E makeE(){
    return new E(){};
  }
}
```
## 传入final变量

## 闭包

## 回调


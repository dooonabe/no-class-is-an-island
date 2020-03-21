# Inner Class
在拥有外部类对象之前不可能创建内部类对象。
```Java
OuterClass out = new OuterClass();
OuterClass.InnerClass = out.new InnerClass();
```

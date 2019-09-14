# Functional Programming Model
## 具有普遍意义的基本构造单元
- Filter(筛选)
- Map(映射)
- Reduce(折叠/化约)

## 记忆与缓求值

## Java 8+
### Method References

Java 8 开始引入了`Function`,`Consumer`,`Supplier`等***函数式接口***，从而将Functional Programming的思想引入到Java语言中。方法（函数）从此可以被当作参数来传递。
默认方法为装饰默认功能提供了便捷途径。
```Java
public interface Function<T, R> {
  R apply(T t);
  
  default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
    Objects.requireNonNull(before);
    return (V v) -> apply(before.apply(v));
  }

  default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
    Objects.requireNonNull(after);
    return (T t) -> after.apply(apply(t));
  }
  
  static <T> Function<T, T> identity() {
    return t -> t;
  }
  
}
```
```Java
public interface Consumer<T> {

  void accept(T t);
  
  default Consumer<T> andThen(Consumer<? super T> after) {
    Objects.requireNonNull(after);
      return (T t) -> { 
        accept(t); 
        after.accept(t); 
      };
  }  

}
```
```Java
public interface Supplier<T> {
  T get();
}
```

SAM与Lambda携手，开发者可以用lambda块取代传统的匿名类来实例化一个接口。

#### 方法引用方式
- 静态方法引用
```Java
Function<String, Integer> converter = Integer::parseInt;
Integer number = converter.apply("10");
```
- 类实例方法引用
```Java
Function<Invoice, Integer> invoiceToId = Invoice::getId;
Integer id = invoiceToId.apply(invoice);
```
- 已存在的类实例方法引用
```Java
Consumer<Object> print = System.out::println;
print.accept(invoice);
```
- 构造方法引用
```Java
Supplier<List<String>> listOfString = List::new;
```

### Anonymous Functions(Lambdas)
- Anonymous


A lambda expression is anonymous because it does not have an explicit name as a method normally would. It’s sort of like ananonymous class in that it does not have a declared name.
- Function

A lambda is like a method in that it has a list of parameters, a body, a return type, and a possible list of exceptions that can be thrown. However, unlike a method, it’s not declared as part of aparticular class.
- Passed around


A lambda expression can be passed as an argument to a method, stored in a variable, and also returned as a result.

lambda表达式也就相当于一个匿名实现类，` @FunctionalInterface`注解说明此接口为函数性接口

### Optional
```Java
public String getCityForEvent(int id) {
  Event event = getEventWithId(id);
    if(event != null) {
      Location location = event.getLocation();
        if(location != null) {
          return location.getCity();
        }
    }
  return "TBC";
}

public String getCityForEvent(int id) {
  Optional.ofNullable(getEventWithId(id))
    .flatMap(this::getLocation)
    .map(this::getCity)
    .orElse("TBC");
}
```

### Functional Programming为Java带来了什么


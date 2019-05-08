# Functional Programming Model

## Java 8+
### Method References
Java 8 开始引入了`Function`,`Consumer`,`Supplier`等基础类，从而将Functional Programming的思想引入到Java语言中。方法（函数）从此可以被当作参数来传递。
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





### Functional Programming为Java带来了什么


# Iterable Iterator
## Iterable
`java.lang.Iterable`
```Java
public interface Iterable<T> {

  Iterator<T> iterator();
  
  default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
  }
  
  default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
  }
}
```

## Iterator
`java.util.Iterator`
```Java
public interface Iterator<E> {
  
  boolean hasNext();
  
  E next();
  
  default void remove() {
        throw new UnsupportedOperationException("remove");
  }
  
  default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
  }
}
```

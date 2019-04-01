# Comparable与Comparator
## 接口待实现方法

### `java.lang.Comparable`
```
public int compareTo(T o)
```

### `java.util.Comparator`

```
int compare(T o1, T o2)
boolean equals(Object obj)
```

## Comparable
类实现了Comparable接口不仅可以用于此类对象间的等同行比较，还可以与泛型算法（generic algorithm）以及依赖于该接口的集合实现（collection implementation）进行协作。
例如：
- 集合工具类Collections的排序方法 
```
public static <T extends Comparable<? super T>> void sort(List<T> list)
```
此方法接收一个实现Comparable的类对象列表，实现对此列表的排序。排序的规则来源于`compareTo`方法，此方法定义的排序规则被称为该类的自然排序（natural ordering）。

## Comparator
Comparator更像是类排序功能的拓展：一个交易指令实体大部分情况下会按照发生时间这个属性进行排序，但是在复盘时也可能会按照指令最终状态属性进行排序，这种情况下按照发生时间属性排序可以被定义为自然排序，实现Comprable接口定义这种排序规则，而最终状态的排序则可以实现Comparator接口定义。


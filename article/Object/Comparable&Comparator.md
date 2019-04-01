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
类实现了Comparable接口不仅可以用于此类对象间的等同性比较，还可以与泛型算法（generic algorithm）以及依赖于该接口的集合实现（collection implementation）进行协作。
例如：
- 集合工具类Collections的排序方法 
```
public static <T extends Comparable<? super T>> void sort(List<T> list)
```
此方法接收一个实现Comparable的类对象列表，实现对此列表的排序。排序的规则来源于`compareTo`方法，此方法定义的排序规则被称为该类的自然排序（natural ordering）。

- 如果一个对象被用于任何种类的排序容器（`TreeMap` `TreeSet`）中，那么它必须实现Comparable接口

## Comparator
Comparator更像类排序功能的拓展。举个例子，一个交易指令实体大部分情况下会按照**发生时间**属性进行排序，但是在复盘时也可能会按照**指令最终状态**属性进行排序，这种情况下按照**发生时间**属性排序可以被定义为自然排序，实现Comprable接口定义这种排序规则，而**指令最终状态**的排序则可以实现Comparator接口定义。
例如：

- 集合工具类Collections的排序方法也支持外部排序规则
```
public static <T> void sort(List<T> list, Comparator<? super T> c)
```
进一步跟入代码发现原来实际使用了数组工具类Arrays的排序方法
```
public static <T> void sort(T[] a, Comparator<? super T> c) {
    if (c == null) {
        sort(a);
    } else {
        if (LegacyMergeSort.userRequested)
            legacyMergeSort(a, c);
        else
            TimSort.sort(a, 0, a.length, c, null, 0, 0);
    }
}
```
而此方法也是上文中`Collections.sort(List<T> list)`方法最终的实现方法。

- 倒置，连续比较
Comparator提供了很多`default`方法，包括`reversed` `thenComparing`等，呼应了java8引入的流式编程理念。

## 注意事项

- `return i1-i2`为常见编程错误。因为int不够大，上述运算可以导致int类型溢出并返回负值，导致程序不正确

- java.lang.String 朴实的compareTo方法实现
```
public int compareTo(String anotherString) {
    int len1 = value.length;
    int len2 = anotherString.value.length;
    int lim = Math.min(len1, len2);
    char v1[] = value;
    char v2[] = anotherString.value;

    int k = 0;
    while (k < lim) {
        char c1 = v1[k];
        char c2 = v2[k];
        if (c1 != c2) {
            return c1 - c2;
        }
        k++;
    }
    return len1 - len2;
}
```

### 参考
- 《JAVA编程思想》
- 《Effective Java》
-  [Java Sorting: Comparator vs Comparable Tutorial](https://www.digizol.com/2008/07/java-sorting-comparator-vs-comparable.html)


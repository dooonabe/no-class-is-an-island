# Map

## Arch
- Map
- ConcurrentMap
- AbstractMap
- HashMap

### LinkedHashMap extends HashMap

LinkedHashMap 怎样记录节点的插入顺序，以及如何实现LRU功能。
`java.util.LinkedHashMap`
```Java
/**
    * The head (eldest) of the doubly linked list.
    */
transient LinkedHashMap.Entry<K,V> head;

/**
    * The tail (youngest) of the doubly linked list.
    */
transient LinkedHashMap.Entry<K,V> tail;

// 扩充HashMap.Node，构造所有节点组成的双向链表
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}

// override HashMap method
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    linkNodeLast(p);
    return p;
}

// link at the end of list
private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    tail = p;
    if (last == null)
        head = p;
    else {
        p.before = last;
        last.after = p;
    }
}
```

### TreeMap

A Red-Black tree based NavigableMap implementation.The map is sorted according to the Comparable (natural ordering) of its keys, or by a Comparator provided at map creation time, depending on which constructor is used.
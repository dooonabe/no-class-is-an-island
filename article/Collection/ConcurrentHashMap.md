# ConcurrentHashMap

## jdk1.7
锁分段技术

## jdk1.8
CAS技术

```Java
// volatile 内存可见 执行屏障
private transient volatile int sizeCtl;

private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    // while + cas 
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0)
            // 有线程已经成功初始化了
            Thread.yield(); // lost initialization race; just spin
        // compareAndSwapInt(内存地址，字段偏移量，字段旧的预期值，字段新值)     
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                // sizeCtl < 0
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}

static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}

final V putVal(K key, V value, boolean onlyIfAbsent) {
    // key value不能为null
    if (key == null || value == null) throw new NullPointerException();
    // 计算hash位
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        // tab是否为空
        if (tab == null || (n = tab.length) == 0)
            // 初始化
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // 该hash位是空，可以直接插入，但是如果不做并发处理会有HashMap线程不安全的问题，所以使用cas插入
            if (casTabAt(tab, i, null,
                            new Node<K,V>(hash, key, value, null)))
                // 成功则break  
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            // 扩容
            tab = helpTransfer(tab, f);
        else {
            // 该hash位不为空，新节点需要在同步的情况下插入
            V oldVal = null;
            // 同步锁细化到某个node节点
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                    (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                            value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                        value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```
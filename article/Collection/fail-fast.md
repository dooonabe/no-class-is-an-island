# 快速报错（fail-fast）机制

Java容器有一种保护机制，它会探查容器上的任何除了你的进程所进行的操作以外的所有变化，一旦它发现其他进程修改了容器，就会立刻抛出`ConcurrentModificationException`异常。这就是“快速报错”的意思——不是使用复杂的算法在事后来检查问题。

### 原理

`java.lang.ArrayList`返回的Iterator对象的实现如下

```Java
private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    int expectedModCount = modCount;

    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        checkForComodification();
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public void forEachRemaining(Consumer<? super E> consumer) {
        Objects.requireNonNull(consumer);
        final int size = ArrayList.this.size;
        int i = cursor;
        if (i >= size) {
            return;
        }
        final Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length) {
            throw new ConcurrentModificationException();
        }
        while (i != size && modCount == expectedModCount) {
            consumer.accept((E) elementData[i++]);
        }
        // update once at end of iteration to reduce heap write traffic
        cursor = i;
        lastRet = i - 1;
        checkForComodification();
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```
观察`checkForComodification`方法，该方法通过比较`modCount` ，`expectedModCount` 这两个成员的值来判断是否有其他线程在容器被遍历的时候操作了容器。

### 其他
`ConcurrentHashMap`， `CopyOnWriteArrayList` ，`CopyOnWriteArraySet` 都使用了可以避免`ConcurrentModificationException`的技术

### 参考
- 《JAVA编程思想》

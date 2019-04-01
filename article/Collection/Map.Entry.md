# Map.Entry

### java.util.Map
```
interface Entry<K,V> {

    V getValue();

    V setValue(V value);

    boolean equals(Object o);

    int hashCode();

    public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K,V>> comparingByKey() {
        return (Comparator<Map.Entry<K, V>> & Serializable)
            (c1, c2) -> c1.getKey().compareTo(c2.getKey());
    }

    public static <K, V extends Comparable<? super V>> Comparator<Map.Entry<K,V>> comparingByValue() {
        return (Comparator<Map.Entry<K, V>> & Serializable)
            (c1, c2) -> c1.getValue().compareTo(c2.getValue());
    }

    public static <K, V> Comparator<Map.Entry<K, V>> comparingByKey(Comparator<? super K> cmp) {
        Objects.requireNonNull(cmp);
        return (Comparator<Map.Entry<K, V>> & Serializable)
            (c1, c2) -> cmp.compare(c1.getKey(), c2.getKey());
    }

    public static <K, V> Comparator<Map.Entry<K, V>> comparingByValue(Comparator<? super V> cmp) {
        Objects.requireNonNull(cmp);
        return (Comparator<Map.Entry<K, V>> & Serializable)
            (c1, c2) -> cmp.compare(c1.getValue(), c2.getValue());
    }
}
```

### java.util.AbstractMap
```
public static class SimpleEntry<K,V> implements Entry<K,V>, java.io.Serializable{
	private static final long serialVersionUID = -8499721149061103585L;
	private final K key;
	private V value;

	public SimpleEntry(K key, V value) {
		this.key   = key;
		this.value = value;
	}

	public SimpleEntry(Entry<? extends K, ? extends V> entry) {
		this.key   = entry.getKey();
		this.value = entry.getValue();
	}

	public K getKey() {
		return key;
	}

	public V getValue() {
		return value;
	}

	public V setValue(V value) {
		V oldValue = this.value;
		this.value = value;
		return oldValue;
	}

	public boolean equals(Object o) {
		if (!(o instanceof Map.Entry))
			return false;
		Map.Entry<?,?> e = (Map.Entry<?,?>)o;
		return eq(key, e.getKey()) && eq(value, e.getValue());
	}
   
	public int hashCode() {
		return (key   == null ? 0 :   key.hashCode()) ^
			   (value == null ? 0 : value.hashCode());
	}

	public String toString() {
		return key + "=" + value;
	}

}
```

### java.util.HashMap
```
static class Node<K,V> implements Map.Entry<K,V> {
	final int hash;
	final K key;
	V value;
	Node<K,V> next;

	Node(int hash, K key, V value, Node<K,V> next) {
		this.hash = hash;
		this.key = key;
		this.value = value;
		this.next = next;
	}

	public final K getKey()        { return key; }
	public final V getValue()      { return value; }
	public final String toString() { return key + "=" + value; }

	public final int hashCode() {
		return Objects.hashCode(key) ^ Objects.hashCode(value);
	}

	public final V setValue(V newValue) {
		V oldValue = value;
		value = newValue;
		return oldValue;
	}

	public final boolean equals(Object o) {
		if (o == this)
			return true;
		if (o instanceof Map.Entry) {
			Map.Entry<?,?> e = (Map.Entry<?,?>)o;
			if (Objects.equals(key, e.getKey()) &&
				Objects.equals(value, e.getValue()))
				return true;
		}
		return false;
	}
}
```

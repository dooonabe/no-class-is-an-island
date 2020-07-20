# equals hashCode

## hashCode
```Java
java.util.HashMap

static final int hash(Object key) {
    int h;
    //^(xor异或)运算规则是:两个数转为二进制，然后从高位开始比较，如果相同则为0，不相同则为1
    //>>>:无符号右移
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

```

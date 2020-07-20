# Pass by Value

we passed is object's reference.

## case1

```Java
public static void main(String[] args) {
    Dog aDog = new Dog("Max");
    Dog oldDog = aDog;

    // we pass the object to foo
    foo(aDog);
    // aDog variable is still pointing to the "Max" dog when foo(...) returns
    aDog.getName().equals("Max"); // true
    aDog.getName().equals("Fifi"); // false
    aDog == oldDog; // true
}

public static void foo(Dog d) {
    d.getName().equals("Max"); // true
    // change d inside of foo() to point to a new Dog instance "Fifi"
    d = new Dog("Fifi");
    d.getName().equals("Fifi"); // true
}
// all primitive arrays are objects , but primitive is not object.
```

方法的调用实际是虚拟机栈(JVMS)顶的栈帧入栈出栈。foo方法执行结束之后，对应的栈帧会出栈，栈顶的栈帧变为main方法,栈帧中的局部变量表中的aDog引用的值还是foo入栈之前的值。


## case2
```Java
public static void main(String[] args) {
    Dog aDog = new Dog("Max");
    Dog oldDog = aDog;

    foo(aDog);
    // when foo(...) returns, the name of the dog has been changed to "Fifi"
    aDog.getName().equals("Fifi"); // true
    // but it is still the same dog:
    aDog == oldDog; // true
}

public static void foo(Dog d) {
    d.getName().equals("Max"); // true
    // this changes the name of d to be "Fifi"
    d.setName("Fifi");
}
```
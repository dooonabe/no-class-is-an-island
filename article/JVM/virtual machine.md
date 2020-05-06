# 虚拟机执行子系统
代码编译结果从`本地机器码`转变为`字节码`，是存储格式发展的一小步，却是编程语言进步的一大步。

## 类加载
有且仅有5种情况会触发类的初始化
1. 遇到`new`, `getstatic`, `putstatic`, `invokestatic`四种字节码时，类会被初始化。被`final`修饰的静态字段在编译器会进行传播优化，因此不会触发类初始化
2. 使用`java.lang.invoke`包的方法对类进行反射调用时
3. 初始化一个类时，其父类还没有进行过初始化时
4. 入口类(main class)
5. `java.lang.invoke.MethodHandle`实例的最后解析结果是对`static`的操作(`REF_getStatic`,`REF_putStatic`,`REF_invokeStatic`)

Difference between Reflection and MethodHandle
```Java
class Test {
    static class ClassA {
        public void println(String s) {
            System.out.println(s);
        }
    }
    
    public static void main(String[] args) throws Throwable {
        // MethodHandle
        Object o = System.currentTimeMillis() % 2 == 0 ? new ClassA() : System.out;
        getPrintLnMH(o).invokeExact("methodHandle");
        //Reflection
        o.getClass().getMethod("println", String.class).invoke(o, "reflection");
    }

    private static MethodHandle getPrintLnMH(Object receiver) throws NoSuchMethodException, IllegalAccessException {
        // 第一个参数是返回值类类型，第二个参数是入参类类型
        MethodType methodType = MethodType.methodType(void.class, String.class);
        return MethodHandles.lookup().findVirtual(receiver.getClass(), "println", methodType).bindTo(receiver);
    }
}
```
1. Reflection模拟Java代码层次的方法调用，MethodHandle模拟字节码层次的方法调用
2. Reflection是重量级，MethodHandle是轻量级
3. MethodHandle理论上可以享受虚拟机的各种优化

总之，Reflection面向的是Java语言，MethodHandle面向的是所有运行在虚拟机上的语言。



### 加载
1. 通过一个类的全限定名来获取定义此类的二进制字节流
2. 将这个字节流所代表的**静态存储结构**转化为**方法区的运行时数据结构**
3. 在内存中生成一个代表这个类的`java.lang.class`对象，作为方法区这个类的访问入口


### 连接(linking)
连接可以细分为验证，准备，解析。
#### 验证

#### 准备
正式为**类变量**分配内存并设置类变量初始值(使用方法区内存)
#### 解析
虚拟机将常量池中的符号引用替换为直接引用。

类或接口的解析过程：

1. 如果符号引用指向的不是数组类型，符号引用所在类的类加载器会加载目标的全限定名
2. 是数组类型并且元素类型为对象，按照第1点加载数组元素类型，接着由**虚拟机**生成一个代表此**数组**维度和元素的数组对象
3. 如果上述步骤没有出现异常，方法区中会有目标的访问接口了，之后还需进行**符号引用验证**(是否可访问，是否存在目标方法或属性)

### 初始化
执行**类构造器**`<clint>()`。类构造器与实例构造器不同，类构造器不需要显示的调用父类的类构造，虚拟机会保证父类的类构造已经执行完毕（在虚拟机中第一个被执行的类构造器是`java.lang.Object`的）。

### 类加载器与双亲委派模型
类加载器分为两种：
- 启动类加载器（Bootstrap ClassLoader），属于JNI
- 其他类加载器，独立于虚拟机，全部继承于java.lang.ClassLoader

`java.land.ClassLoader`
```Java
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException{
    synchronized (getClassLoadingLock(name)) {
        // First, check if the class has already been loaded
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }

            if (c == null) {
                // If still not found, then invoke findClass in order
                // to find the class.
                long t1 = System.nanoTime();
                c = findClass(name);

                // this is the defining class loader; record the stats
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}
```

## 虚拟机字节码执行引擎
### 运行时栈帧结构
1. 局部变量表
2. 操作数栈
3. 动态连接
4. 方法返回地址

### 分派
外观类型(apparent type) VS 实际类型(actual type)

## 案例

### Tomcat
要解决的问题：
- 不同应用使用的Java类库可以相互隔离，例如不同应用使用不同版本的第三方库
- 不同应用使用的Java类库可以互相共享，避免同一个类存在多个副本，导致方法区过度膨胀
- 服务器需要保证自身的安全不受部署的应用影响
 
A：如果有10个Web应用程序都是用Spring来进行组织和管理的话，可以把Spring类库放到Common或Shared目录下让这些程序共享。Spring要对用户程序的类进行管理，自然要能访问到用户程序的类，而用户的程序显然是放在/WebApp/WEB-INFO目录中的，那么被CommonClassLoader或SharedClassLoader加载的Spring如何访问并不在其加载范围内的用户程序呢？


Q：创建加载用户程序/WebApp/WEB-INFO目录中类的加载器，并将此加载器的parent属性设置为CommonClassLoader或SharedClassLoader。


### 字节码生成技术用于动态代理的实现
```Java
public class DynamicProxyTest {

    interface IHello<T>{
        T sayHello();
    }

    static class Hello implements IHello<String>{
        @Override
        public String sayHello() {
            System.out.println("World");
            return "World";
        }
    }

    static class DynamicProxy implements InvocationHandler{
        Object originalObj;

        Object bind(Object o){
            this.originalObj = o;
            // 生成了class文件，触发此class文件的类加载流程
            return Proxy.newProxyInstance(originalObj.getClass().getClassLoader(), originalObj.getClass().getInterfaces(), this);
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            return method.invoke(originalObj, args);
        }

    }
    public static void main(String[] args) {
        IHello hello = (IHello) new DynamicProxy().bind(new Hello());
        hello.sayHello();
    }
}

```
### 远程执行功能
在服务中执行一段代码，定位或排除问题。

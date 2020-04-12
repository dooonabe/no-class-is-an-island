# 虚拟机执行子系统
代码编译结果从`本地机器码`转变为`字节码`，是存储格式发展的一小步，却是编程语言进步的一大步。

## 类加载
有且仅有5种情况会触发类的初始化
1. 遇到`new`, `getstatic`, `putstatic`, `invokestatic`四种字节码时，类会被初始化。被`final`修饰的静态字段在编译器会进行传播优化，所以因此触发类初始化
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
### 初始化

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
## 案例

### Tomcat
要解决的问题：
- 不同应用使用的Java类库可以相互隔离，例如不同应用使用不同版本的第三方库
- 不同应用使用的Java类库可以互相共享，避免同一个类存在多个副本，导致方法区过度膨胀
- 服务器需要保证自身的安全不受部署的应用影响
 
A：如果有10个Web应用程序都是用Spring来进行组织和管理的话，可以把Spring类库放到Common或Shared目录下让这些程序共享。Spring要对用户程序的类进行管理，自然要能访问到用户程序的类，而用户的程序显然是放在/WebApp/WEB-INFO目录中的，那么被CommonClassLoader或SharedClassLoader加载的Spring如何访问并不在其加载范围内的用户程序呢？


Q：创建加载用户程序/WebApp/WEB-INFO目录中类的加载器，并将此加载器的parent属性设置CommonClassLoader或SharedClassLoader。


### 字节码生成技术于动态代理的实现
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

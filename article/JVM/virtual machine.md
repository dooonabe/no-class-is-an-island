# 虚拟机执行子系统
代码编译结果从`本地机器码`转变为`字节码`，是存储格式发展的一小步，却是编程语言进步的一大步。

## 类加载
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
- 不同应用使用的Java类库可以相互隔离，不同版本的第三方库
- 不同应用使用的Java类库可以互相共享，避免不同应用的方法区过度膨胀
- 服务器需要保证自身的安全不受部署的应用影响
 
A：如果有10个Web应用程序都是用Spring来进行组织和管理的话，可以把Spring放到Common或Shared目录下让这些程序共享。Spring要对用户程序的类进行管理，自然要能访问到用户程序的类，而用户的程序显然是放在/WebApp/WEB-INFO目录中的，那么被CommonClassLoader或SharedClassLoader加载的Spring如何访问并不在其加载范围内的用户程序呢？



Q：创建加载用户程序/WebApp/WEB-INFO目录中类的加载器，并将此加载器的parent属性设置CommonClassLoader或SharedClassLoader。

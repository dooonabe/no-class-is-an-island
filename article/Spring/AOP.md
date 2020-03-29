# AOP
切入点（pointcut）与通知（advice）两者构成切面（aspect）
## Aspect与Ajc编译器
静态织入：AspectJ主要采用的是编译期织入，编译期时AspectJ的acj编译器把aspect类编译成class字节码后，在java目标类编译时织入，即先编译aspect类再编译目标类。
## Spring AOP
动态织入：Spring AOP采用的就是基于运行时增强的代理技术，实现分为JDK的动态代理(InvocationHandler,底层通过反射实现)和CGLIB的动态代理(底层通过继承实现)。

1. JDK动态代理：

```Java
package java.lang.reflect.InvocationHandler

public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args)
        // 如何处理这些方法
        throws Throwable;
}

package java.lang.reflect.Proxy

public class Proxy implements java.io.Serializable {

    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
    ...
    }

} 
```
从创建代理实例的方法参数中可以看到，想要使用代理必须要有实现某个接口类。

2. CGLIB
```Java
public class CglibProxy implements MethodInterceptor {
    private BookApplication application;

    public CglibProxy(BookApplication application) {
        this.application = application;
    }

    public BookApplication createProxy(){
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(application.getClass());
        enhancer.setCallback(this);
        return (BookApplication) enhancer.create();
    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        // aop...
        return methodProxy.invokeSuper(o, objects);
    }
}
```





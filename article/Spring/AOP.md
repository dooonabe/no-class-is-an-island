# AOP
切入点（pointcut）与通知（advice）两者构成切面（aspect）
## Aspect与Ajc编译器
静态织入：ApectJ主要采用的是编译期织入，编译期时AspectJ的acj编译器把aspect类编译成class字节码后，在java目标类编译时织入，即先编译aspect类再编译目标类。
## Spring AOP
动态织入：Spring AOP采用的就是基于运行时增强的代理技术，实现分为Java JDK的动态代理(Proxy，底层通过反射实现)和CGLIB的动态代理(底层通过继承实现)。

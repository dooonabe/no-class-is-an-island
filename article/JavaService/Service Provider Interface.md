# SPI
Service Provider Interface, 服务提供方接口。
```Java
java.util.ServiceLoader

public final class ServiceLoader<S>
    implements Iterable<S>
{

    private static final String PREFIX = "META-INF/services/";

    // The class or interface representing the service being loaded
    private final Class<S> service;

    // The class loader used to locate, load, and instantiate providers
    private final ClassLoader loader;

    // The access control context taken when the ServiceLoader is created
    private final AccessControlContext acc;

    // Cached providers, in instantiation order
    private LinkedHashMap<String,S> providers = new LinkedHashMap<>();

    // The current lazy-lookup iterator
    private LazyIterator lookupIterator;

    ......
}
```

## SLF4J
SLF4J如何找到实现类?

## JDBC
谁触发了JDBC实现类的加载?

Before JDK1.6: `Class.forName("com.mysql.cj.jdbc.Driver")`

After JDK1.6: `java.util.ServiceLoader`

## DUBBO

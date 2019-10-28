# Bean

Spring框架关注于通过DI,AOP和消除样板式代码来简化Java开发。

## Bean的生命周期

### InitializingBean and DisposableBean callback interfaces

```Java
public interface InitializingBean {
    void afterPropertiesSet() throws Exception;
}

public interface DisposableBean {
    void destroy() throws Exception;
}

```
### *Aware interfaces for specific behavior

### Custom init() and destroy() methods in bean configuration file

自定义初始化方法与自定义销毁方法
### @PostConstruct and @PreDestroy annotations

自定义初始化方法与自定义销毁方法

## 参考
- [spring-bean-life-cycle](https://howtodoinjava.com/spring-core/spring-bean-life-cycle/)

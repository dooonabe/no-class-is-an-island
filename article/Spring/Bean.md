# Bean

Spring框架关注于通过DI,AOP和消除样板式代码来简化Java开发。

## Bean的作用域

Spring定义了多种作用域，可以基于这些作用域创建bean。
- 单例（Singleton）：在整个应用中，只创建bean的一个实例。 
- 原型（Prototype）：每次注入或者通过Spring应用上下文获取的时候，都会创建一个新的bean实例。 
- 请求（Rquest）：在Web应用中，为每个请求创建一个bean实例。
- 会话（Session）：在Web应用中，为每个会话创建一个bean实例。
- 全局会话（global-session）：在Web应用中，为全局会话创建一个bean实例。

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
```Java
public interface BeanNameAware extends Aware {
    void setBeanName(String var1);
}

```
### Custom init() and destroy() methods in bean configuration file

自定义初始化方法与自定义销毁方法
### @PostConstruct and @PreDestroy annotations

自定义初始化方法与自定义销毁方法

## 参考
- [spring-bean-life-cycle](https://howtodoinjava.com/spring-core/spring-bean-life-cycle/)

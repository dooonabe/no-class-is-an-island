# Spring IOC

## 低耦合，低侵入
对象间的组合使用是常态，Spring IOC的设计目的是降低对象间的耦合度，以及代码的侵入度。

## 依赖注入，控制反转

### 手动装配与注入
#### Setter注入
Setter注入的原理是通过属性的set方法，为属性赋值。


例如配置类中的属性字段值注入：
```Java
application.properties
spring.redis.clusterNodes[0]=test1
spring.redis.clusterNodes[1]=test1
spring.redis.clusterNodes[2]=test1
spring.redis.port=6379
spring.redis.timeout=1000


@ConfigurationProperties(prefix = "spring.redis")
@Component
public class RedisConfig {
    private List<String> clusterNodes;
    private String port;
    private String timeout;

    public List<String> getClusterNodes() {
        return clusterNodes;
    }

    public void setClusterNodes(List<String> clusterNodes) {
        this.clusterNodes = clusterNodes;
    }

    public String getPort() {
        return port;
    }

    public void setPort(String port) {
        this.port = port;
    }

    public String getTimeout() {
        return timeout;
    }

    public void setTimeout(String timeout) {
        this.timeout = timeout;
    }
}

```
#### 构造器注入

### 自动装配与注入
#### byType
#### byName
#### constructor




## 参考
- [关于Spring IOC (DI-依赖注入)你需要知道的一切](https://blog.csdn.net/javazejian/article/details/54561302)

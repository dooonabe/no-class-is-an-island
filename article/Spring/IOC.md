# Spring IOC

## 低耦合，低侵入
对象间的组合使用是常态，Spring IOC的设计目的是降低对象间的耦合度，以及代码的侵入度。

## 依赖注入，控制反转

### 装配与注入
### 装配
- 定义名叫redisClient的bean
```Java
@Configuration
public class BeanConfiguration {

    @Bean
    public JedisCluster redisClient(){
        return new JedisCluster();
    }
}
```
@Configuration注解标明BeanConfiguration类。注解@Configuration等价于＜beans＞标签。在该类中，每个使用注解@Bean的公共方法对应着一个＜bean＞标签的定义，即@Bean等价于＜bean＞标签。这种基于java的注解配置方式是在spring3.0中引入的。

- 根据配置文件装配bean
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
- 使用bean
```Java
@Test
public void testByConfigurationAnnotation() throws Exception {
    AnnotationConfigApplicationContext config=new AnnotationConfigApplicationContext(BeanConfiguration.class);
    //名称必须与BeanConfiguration中工程方法名称一致
    JedisCluster jedisCluster= (JedisCluster) config.getBean("redisClient");
}
```
### 手动注入
#### Setter注入
Setter注入的原理是通过属性的set方法，为属性赋值。


`<property></property>`
#### 构造器注入
`<constructor-arg></constructor-arg>`

### 自动注入
#### byType
当存在多个相同类型的bean是，此方法会失败。
通过@Autowired的使用标注到成员变量时不需要有set方法，@Autowired 默认按类型匹配。

#### byName
如果@Autowired需要按名称(byName)匹配的话，可以使用@Qualifier注解与@Autowired结合，请注意必须在xml配置中启动注解驱动。
Spring容器对于@Resource注解的name属性解析为bean的名字，type属性则解析为bean的类型。因此使用name属性，则按byName模式的自动注入策略，如果使用type属性则按 byType模式自动注入策略。倘若既不指定name也不指定type属性，Spring容器将通过反射技术默认按byName模式注入。
#### constructor




## 参考
- [关于Spring IOC (DI-依赖注入)你需要知道的一切](https://blog.csdn.net/javazejian/article/details/54561302)

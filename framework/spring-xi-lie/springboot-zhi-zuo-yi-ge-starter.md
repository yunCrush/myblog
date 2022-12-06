---
description: 制作一个简单的starter，这里只是记录一个流程
---

# SpringBoot制作一个Starter

### 1. 前提

1. 注解

{% code overflow="wrap" %}
```yaml
@Bean：声明这是一个对象，可以指定名称，默认是方法名
@Configuration：表明当前类是一个配置类，配合@Bean注解将对象注入Spring容器类
@ConditionalOnClass(XX.class)：在类路径下存在XX类时，才会构建这个Bean
@EnableConfigurationProperties(SimpleBean.class)：使@ConfigurationProperties(prefix = "simplebean")
@ConfigurationProperties(prefix = "simplebean")：配置文件中simplebean开头的属性注入到以下属性
```
{% endcode %}

### 2.定义一个实体类

SimpleBean: int id, String name;

{% code title="SimpleBean.java" overflow="wrap" %}
```java
// 开启下面的注解@ConfigurationProperties
@EnableConfigurationProperties(SimpleBean.class)
// 配置文件中simplebean开头的属性注入到以下属性
@ConfigurationProperties(prefix = "simplebean")
@Data
public class SimpleBean {
    private int id;
    private String name;

    @Override
    public String toString() {
        return "SimpleBean{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}

```
{% endcode %}

### 3.定义一个配置类，注入Bean

{% code title="MyAutoConfiguration.java" overflow="wrap" %}
```java
@Configuration
/**
 *类路径下存在指定的类时，就会进行自动配置
 * @ConditionalOnClass(SimpleBean.class)
 */
@ConditionalOnClass(SimpleBean.class)
public class MyAutoConfiguration {
    static {
        System.out.println("MyAutoConfiguration init .....");
    }va
    @Bean
    public SimpleBean simpleBean() {
        return new SimpleBean();
    }
}

```
{% endcode %}

### 4.创建spring.factories

spring.factories是SpringBoot的SPI机制，本身Starter就是一个插件类型的。在resources下创建`META-INF/spring.factories`这里内容存放是自动装配的配置类的全路径

文件内容：

{% code title="spring.factories" overflow="wrap" %}
```factor
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.yun.config.MyAutoConfiguration
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (1) (4).png" alt=""><figcaption><p>项目结构图</p></figcaption></figure>

### 5.如何食用这个Starter呢？

1. pom文件中引入依赖

```
 <dependency>
            <groupId>com.yun</groupId>
            <artifactId>yuncrush-spring-boot-starter</artifactId>
            <version>1.0-SNAPSHOT</version>
</dependency>
```

2\. application.yml进行配置

```
simplebean.id=1
simplebean.name=yuncrus
```

3\. 使用@Resource注解使用这个Bean

```
 @Resource
    SimpleBean simpleBean;
```

项目完整地址：[https://github.com/yunCrush/yuncrushspringbootstarter](https://github.com/yunCrush/yuncrushspringbootstarter)

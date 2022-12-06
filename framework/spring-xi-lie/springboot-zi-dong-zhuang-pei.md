# SpringBoot自动装配

　　自动装配：就是将我们需要用到的类（pom.xml文件中配置到的），自动进行配置到bean容器当中去，其中涉及到注解@OnCondition()这个注解，只有在满足一定条件时，会进行相关类的配置，这里面涉及到java 的SPI机制，在META-INF/spring.factories配置了实现类，具体要配置哪个实现类，会根据

1.  开启注解`@EnableAutoConfiguration`，由`@SpringBootApplication`开启的，`@EnableAutoConfiguration`的作用是其中的`@Import` 来引入装配实现，`@AutoConfigurationImportSelector` 将我们的一些处理逻辑加载进来，加载 `META-INF/spring.factories`  配置从其中获取到已有的自动装配，获得一个Key-value的结构，key是 EnableAutoConfiguration，value是string数组，这里体现了**自动**,数组中存的是需要执行的\*AutoConfiguration，执行\*AutoConfiguration才是自动装配的重点，具体自动装配类是否会生效，取决于注解`@ConditionOnClass()` 配置的条件表达式，这里体现了**智能**&#x20;


# SpringBoot自动装配

　　开启注解`@EnableAutoConfiguration`，由`@SpringBootApplication`开启的，`@EnableAutoConfiguration`的作用是其中的`@Import` 来引入装配实现，`@AutoConfigurationImportSelector` 将我们的一些处理逻辑加载进来，加载 `META-INF/spring.factories`  配置从其中获取到已有的自动装配，获得一个Key-value的结构，key是 EnableAutoConfiguration，value是string数组，这里体现了**自动**,数组中存的是需要执行的\*AutoConfiguration，执行\*AutoConfiguration才是自动装配的重点，具体自动装配类是否会生效，取决于注解`@ConditionOnClass()` 配置的条件表达式，这里体现了**智能**&#x20;


# AOP

AOP讲的是关注点分离，对核心业务以及非核心业务，如日志的分离。

横切：对哪些方法进行拦截，拦截后怎么处理，叫横切点关注。

通知：这里拿日志举例，就是记录日志的逻辑，编写好模块，方便直接调用。

连接点：所有的目标方法都可以是连接点，一般是方法的前后

切入点：通过切入点表达式过滤出的一组方法

切面：通知+切入点，在哪里干什么

### 实践AOP

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

```
package com.yc.web.aspect;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

/**
 * Author: yunCrush
 * Date:2023-09-23 23:26
 * Description:
 */
@Component
@Aspect
public class MethodChecker {
    //切入点 表达式：任意返回类型 包路径.任意类.任意方法.任意参数
    @Pointcut("execution(* com.yc.web.controller.*.*(..))")
    public void check() {}


    //通知
    @Before("check()")
    public void beforeCheck() {
        System.out.println("beforeAdvice...");
    }

    @After("check()")
    public void afterCheck() {
        System.out.println("afterAdvice...");
    }

    @Around("check()")
    public void aroundCheck(ProceedingJoinPoint pjp) {
        System.out.println("aroundAdvice...before");
        try {
            pjp.proceed();
        } catch (Throwable e) {
            throw new RuntimeException(e);
        }
        System.out.println("aroundAdvice...after");
    }
}
```

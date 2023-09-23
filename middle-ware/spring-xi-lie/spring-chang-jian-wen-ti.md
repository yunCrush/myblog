---
description: 面试中常见的问题。
---

# Spring常见问题

### 1. 如何解决循环依赖的问题

在IOC容器进行对象创建的时候，是通过构造函数来创建了一个对象A，这时还没有进行属性设置，在进行属性设置时，发现引用了其他对象B，其他B对象的属性又是当前对象A，这样造成了循环依赖。

解决方案：通过构造函数实例化了一个对象，但是此时还么有进行属性赋值，将其丢到**earlySingletonObjects**中

singletonObjects：第一级缓存，里面放置的是实例化好的单例对象；

earlySingletonObjects：第二级缓存，里面存放的是提前曝光的单例对象；

singletonFactories：第三级缓存，里面存放的是要被实例化的对象的对象工厂

从二级缓存中可以获取到，然后将当前对象进行实例化完成，再回退完成对象的实例化。

因此可以通过set方法注入,在set方法之前执行了构造方法，将半成品单例对象丢入earlySingletonObjects

![三级缓存](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/spring-%E4%B8%89%E7%BA%A7%E7%BC%93%E5%AD%98.png)

\


### 2.依赖注入的几种方式

属性注入

构造方法注入

set方法注入，set方法注入可以避免循环依赖问题

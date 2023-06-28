---
layout:
  title:
    visible: false
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 抽象类与接口

　　抽象类的目的主要是代码的重用，可以有一个或者多个抽象方法，抽象类大多用于抽取相关类的共同方法或者共同成员变量，然后通过继承的方式达到代码重用。

　　接口是对行为的抽象，是抽象方法的集合，不可实例化，任何field都隐含这`public static final`

　　抽 象类可以只重写部分方法，而接口需要全部重写

　　Java8以后接口也可以有默认实现了，重写接口方法是可根据需要选择是否对接口的默认实现进行重写。

```
public interface Study {
    // 必须default，才可有默认实现
    default   void study0(){
         System.out.println("love study");
     }
     void test();
}
```

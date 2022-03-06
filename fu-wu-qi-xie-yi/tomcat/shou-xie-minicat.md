---
description: 简单实现一个mini tomcat
---

# 手写minicat

## minicat功能

　　Minicat作为⼀个服务器软件提供服务的，也即我们可以通过浏览器客户端发送http请求， Minicat可以接收到请求进⾏处理，处理之后的结果可以返回浏览器客户端。&#x20;

1）提供服务，接收请求（Socket通信）&#x20;

2）请求信息封装成Request对象（Response对象）&#x20;

3）客户端请求资源，资源分为静态资源（html）和动态资源（Servlet）

&#x20;4）资源返回给客户端浏览器

## V1.0

　　需求：浏览器请求http://localhost:8080,返回⼀个固定的字符串到⻚⾯"Hello Minicat!"

## V2.0

　　需求：封装Request和Response对象，返回html静态资源⽂件

## V3.0

　　需求：可以请求动态资源（Servlet）

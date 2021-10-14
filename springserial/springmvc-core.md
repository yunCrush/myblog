---
description: 这里介绍SpringMVC的核心组件，以及SpringMVC的请求流程。
---

# SpringMVC核心组件

## 1. MVC区别三层架构

1.  三层架构

    视图层View: 接收用户请求

    服务层Service: 业务逻辑

    持久层Dao: 操作数据库
2.  MVC

    Model模型：模型，承载数据，对用户的请求进行计算的模块

    View视图：与用户直接交互

    Controller控制器：用于将用户的请求转发给相应的model进行处理

## 2. SpringMVC核心组件

1. DispatcherServlet：前置控制器
2. Handler：继前置控制之后的后端处理器
3. HandlerMapping：将请求映射到Handler
4. HandlerInterceptor：处理器拦截器
5. HandlerExecutionChain：处理器执行链
6. HandlerAdaptor：处理器适配器
7. ModelAndView：装载模型数据和视图
8. ViewResolver：视图解析器

## 3. 执行流程

流程如图所示：

![springmvc流程图](<../.gitbook/assets/image (6).png>)

　　流程：DispatcherServlet前置控制器收到来自客户端的请求，通过HandlerMapping根据请求的url，找到具体的处理器，生成处理器执行链(Handler+处理器拦截器)返回给DispatcherServlet，前置控制器通过处理器Handler获取处理器适配器，执行处理器适配器的一系列操作(数据格式转换，参数封装)，执行处理器Handler执行完后返回ModelAndView模型数据给DispatcherServlet，DispatcherServlet将模型数据给ViewResolver视图解析器进行解析，将解析后的具体的view返回给DispatcherServlet，由DispatcherServlet进行模型的填充与渲染，最后响应给客户端。

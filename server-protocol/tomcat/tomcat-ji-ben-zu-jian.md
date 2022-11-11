---
description: tomcat是一个Http服务器，也是一个Servlet容器
---

# Tomcat安装与简单入门

## 安装部署

```shell
/data/soft/apache-tomcat-8.5.50.tar
tar tar -xvf apache-tomcat-8.5.50.tar
cd /data/soft/apache-tomcat-8.5.50/

conf/
# 日志相关
logging.properties 
# 8080端口的指定等
server.xml
# 跳转到index.html就是配置在web.xml中的
web.xml

# 项目发布相关
webapps/

# JSP 编译运行会产生过程文件
work/

```

## 1.基础理论

　　tomcat的两个核心组件：连接器Connector（http服务器功能，与socket进行通信),Container容器（负责处理内部的请求，加载和管理servlet，以及处理具体的Request).tomcat默认支持http 1.1。自8.5以后支持http 2.0协议。传输层支持I/O模型：NIO/NIO2/APR。其中APR模型：Tomcat将以JNI的形式调用Apache HTTP服务器的核心动态链接库来处理文件读取或网络传输操作。

![tomcat component](<../../.gitbook/assets/image (42) (1).png>)

![connector and container](<../../.gitbook/assets/image (38).png>)

## 2.核心组件Coyote

　coyote是tomcat中连接器的名称，对外的接口，客户端与coyote进行交互，建立连接，处理响应请求。

coyote包括如下部分:endPoint(coyote的通信端点,实现TCP协议/IP),Processor(实现Http协议，是对应用层的抽象), Adapter(用于将上图中的request请求封装成ServletRequest，以及ServletResponse-> Reponse,**适配器的经典使用**)。

![connector](<../../.gitbook/assets/image (40) (1).png>)

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>protocol</p></figcaption></figure>

## 3.核心组件Catalina

　tomcat本质就是一个servlet容器，是一个catalina实例，故catalina才是tomcat核心。

下图一个Server对应多个service,但是一般都是一个，对应多个connector连接器，多个connector对应一个container容器。

![tomcat 实例](<../../.gitbook/assets/image (36).png>)

1. Catalina 负责解析Tomcat的配置⽂件（server.xml） , 以此来创建服务器Server组件并进⾏管理
2. Server 服务器表示整个Catalina Servlet容器以及其它组件，负责组装并启动Servlaet引擎,Tomcat连接 器。Server通过实现Lifecycle接⼝，提供了⼀种优雅的启动和关闭整个系统的⽅式
3. Service 服务是Server内部的组件，⼀个Server包含多个Service。它将若⼲个Connector组件绑定到⼀个 Container
4. Container 容器，负责处理⽤户的servlet请求，并返回对象给web⽤户的模块

**container具体结构**：

1. Engine 表示整个Catalina的Servlet引擎，⽤来管理多个虚拟站点，⼀个Service最多只能有⼀个Engine， 但是⼀个引擎可包含多个Host
2. Host 代表⼀个虚拟主机，或者说⼀个站点，可以给Tomcat配置多个虚拟主机地址，⽽⼀个虚拟主机下 可包含多个Context,比如监听的8080端口，可以在server.xml下配置多个host, localhost:8080/a和localhost:8080/b
3. Context 表示⼀个Web应⽤程序， ⼀个Web应⽤可包含多个Wrapper
4. Wrapper 表示⼀个Servlet，Wrapper 作为容器中的最底层，不能包含⼦容器

具体配置都体现在server.xml中

## 4.核心配置文件server.xml

　　核心配置文件server.xml存在conf/server.xml

主要标签结构：

```
<!--
 Server 根元素，创建⼀个Server实例，⼦标签有 Listener、GlobalNamingResources、Service
-->
<Server>
   <!--定义监听器-->
   <Listener/>
   <!--定义服务器的全局JNDI资源 -->
   <GlobalNamingResources/>
 <!--定义⼀个Service服务，⼀个Server标签可以有多个Service服务实例-->
     <Service name="Catalina">
    <!--The connectors can use a shared executor, you can define one or more named thread pools-->
    <!-- 可以配置一个线程池来处理下面的connector连接，但是默认是关闭的。
    <Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
        maxThreads="150" minSpareThreads="4"/>
    -->
</Server>
```

1. **connector 标签**

```
# service下connector标签
 <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
# port： 端⼝号，Connector ⽤于创建服务端Socket 并进⾏监听， 以等待客户端请求链接。
如果该属性设置为0， Tomcat将会随机选择⼀个可⽤的端⼝号给当前Connector使⽤
# protocol：默认Http/1.1协议
# connectionTimeout:连接超时，-1，表示永不超时。
# redirectPort：当前Connector 不⽀持SSL请求， 接收到了⼀个请求， 
并且也符合security-constraint 约束，需要SSL传输，Catalina⾃动将请求重定向到指定的端⼝。
```

**2. engine 标签**

```
# engine标签，只有一个，defaultHost表示仅仅处理localhost://...
<Engine name="Catalina" defaultHost="localhost">
```

**3. host标签**

```
#  Host标签  
   <Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
# appBase: localhost 请求到了去webapps/ROOT找相关资源，正常都是war包，所以配置
unpackWARS="true",资源有一定变更时，自动部署。
```

4\. **配置connector共享线程池Executor标签**

```
 <Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
        maxThreads="150" minSpareThreads="4"/>
# 线程名称:namePrefix+number
=================================================================================
默认情况下，Service 并未添加共享线程池配置。 如果我们想添加⼀个线程池， 可以在
<Service> 下添加如下配置：
 name：线程池名称，⽤于 Connector中指定
 namePrefix：所创建的每个线程的名称前缀，⼀个单独的线程名称为
 namePrefix+threadNumber
 maxThreads：池中最⼤线程数
 minSpareThreads：活跃线程数，也就是核⼼池线程数，这些线程不会被销毁，会⼀直存在
 maxIdleTime：线程空闲时间，超过该时间后，空闲线程会被销毁，默认值为6000（1分钟），单位
毫秒
 maxQueueSize：在被执⾏前最⼤线程排队数⽬，默认为Int的最⼤值，也就是⼴义的⽆限。除⾮特
殊情况，这个值 不需要更改，否则会有请求不会被处理的情况发⽣
 prestartminSpareThreads：启动线程池时是否启动 minSpareThreads部分线程。默认值为
false，即不启动
 threadPriority：线程池中线程优先级，默认值为5，值从1到10
 className：线程池实现类，未指定情况下，默认实现类为
org.apache.catalina.core.StandardThreadExecutor。如果想使⽤⾃定义线程池⾸先需要实现
org.apache.catalina.Executor接⼝
==============================================================================
<Executor name="commonThreadPool"
 namePrefix="thread-exec-"
 maxThreads="200"
 minSpareThreads="100"
 maxIdleTime="60000"
 maxQueueSize="Integer.MAX_VALUE"
 prestartminSpareThreads="false"
 threadPriority="5"
 className="org.apache.catalina.core.StandardThreadExecutor"/>
```

## 5. 配置两个虚拟主机host

```
# 复制一份webapps,存放不同资源文件
[root@VM-0-15-centos apache-tomcat-8.5.50]# cp -r webapps webapps2
#修改server.xml与 webapps2/ROOT/index.jsp
```

![webapps2/ROOT/index.jsp](<../../.gitbook/assets/image (43).png>)

![server.xml](<../../.gitbook/assets/image (41) (1).png>)

通过访问不同的url即可得到不同的响应页面

url1: http://www.abc.om:8080/

url2: http://www.def.com:8080/

## 6.Context配置特定资源

```
# 访问：http://www.abc.om/web3 寻找docBase下的相关文件
# 对于不同的相关文件可以通过不同的context进行配置
<Host name="www.abc.com" appBase="webapps" unpackWARs="true" autoDeploy="true">
<!--
 docBase：Web应⽤⽬录或者War包的部署路径。可以是绝对路径，也可以是相对于 Host appBase的
相对路径。
 path：Web应⽤的Context 路径。如果我们Host名为localhost， 则该web应⽤访问的根路径为：
 http://localhost:8080/web3。
-->
 <Context docBase="/Users/yingdian/web_demo" path="/web3"></Context>

# 日志相关
 <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
 prefix="localhost_access_log" suffix=".txt"
 pattern="%h %l %u %t &quot;%r&quot; %s %b" />
</Host>
```

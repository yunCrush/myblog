---
description: 优点：配置简单，高性能，稳定性好， 宕机概率低。
---

# Nginx安装及基本配置

## 1.安装

```
# 安装Nginx依赖，pcre、openssl、gcc、zlib（推荐使⽤yum源⾃动安装）
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
# 解包Nginx软件包
tar -xvf nginx-1.17.8.tar
# 进⼊解压之后的⽬录 nginx-1.17.8
cd nginx-1.17.8
# 命令⾏执⾏./configure
# 命令⾏执⾏ make
# 命令⾏执⾏ make install，完毕之后在/usr/local/下会产⽣⼀个nginx⽬录
```

## 2.基本概念

反向代理：请求到nginx服务器，client无法感知server端，请求到nginx,隐藏了server端，由nginx进行请求转发到tomcat端，nginx对client而言即服务端。

正向代理：对server端tomcat而言，无法感知client端，client端请求到达一个服务器，服务器再将请求转发到tomcat,服务器对tomcat而言即客户端。

动静分离：nginx适合存放静态资源，tomcat处理动态请求。

负载均衡：client请求到达nginx,由Nginx进行判断将请求转发到哪个服务器tomcat。

```
# 配置文件conf/nginx.conf 启动脚本sbin/nginx
./sbin/nginx -s reload 重新加载配置文件
./sbin/nginx -s stop 
```

## 3.配置文件

配置文件由三部分构成：全局块，events块，http块。

3.1 全局块：从配置⽂件开始到events块之间的内容，此处的配置影响nginx服务器整体的运⾏，⽐如worker进 程的数量、错误⽇志的位置等。

![全局块](<../../.gitbook/assets/image (38) (1).png>)

3.2 events块：events块主要影响nginx服务器与⽤户的⽹络连接，⽐如worker\_connections 1024，标识每个 workderprocess⽀持的最⼤连接数为1024。

![events块](<../../.gitbook/assets/image (37) (1).png>)

3.3 http块：http块是配置最频繁的部分，虚拟主机的配置，监听端⼝的配置，请求转发、反向代理、负载均衡 等，一个http下的server有多个localtion。

![](<../../.gitbook/assets/image (40) (1) (1).png>)

![](<../../.gitbook/assets/image (36) (1).png>)

例子：反向代理-负载均衡：localhost:9003/abc -->跳转到8080或者8082端口。

负载均衡策略：1. 默认轮询。2. 权重，在端口后添加 weight=2; 权重高分配请求多。3. ip\_hash;ip在获取到hash值后，可将每次的请求都转发到一个服务器。

动静分离：localhost:9003/static/abc.html -->跳转到nginx/staticData/static/abc.html

## 4. 底层进程机制

Nginx启动后，以daemon多进程⽅式在后台运⾏，包括⼀个Master进程和多个Worker进程，Master 进程是领导，是⽼⼤，Worker进程是⼲活的⼩弟。

master进程： 主要是管理worker进程，⽐如： 接收外界信号向各worker进程发送信号(./nginx -s reload) 监控worker进程的运⾏状态，当worker进程异常退出后Master进程会⾃动重新启动新的 worker进程等。与全局块配置的worker数量有关。

worker进程：具体处理⽹络请求，多个worker进程之间是对等的，他们同等竞争来⾃客户端的请 求，各进程互相之间是独⽴的。⼀个请求，只可能在⼀个worker进程中处理，⼀个worker进程， 不可能处理其它进程的请求。worker进程的个数是可以设置的，⼀般设置与机器cpu核数⼀致。

```
 ./nginx -s reload 来说明nginx信号处理这部分
1）master进程对配置⽂件进⾏语法检查
2）尝试配置（⽐如修改了监听端⼝，那就尝试分配新的监听端⼝）
3）尝试成功则使⽤新的配置，新建worker进程
4）新建成功，给旧的worker进程发送关闭消息
5）旧的worker进程收到信号会继续服务，直到把当前进程接收到的请求处理完毕后关闭
所以reload之后worker进程pid是发⽣了变化的
```

我们监听9003端⼝，⼀个请求到来时，如果有多个worker进程，那么每个worker进程都有 可能处理这个链接。

```
master进程创建好后，会建立需要监听的socket,然后master fork多个worker,所有的
worker进程的监听描述符listenfd在新连接到来时将变得可读。
nginx使⽤互斥锁来保证只有⼀个workder进程能够处理请求，拿到互斥锁的那个进程注册 
listenfd读事件，在读事件⾥调⽤accept接受该连接，然后解析、处理、返回客户端。
```

nginx最大并发连接：work_process \* work\_connections。如果用作反向代理则需要除以4._

![](<../../.gitbook/assets/image (41) (1).png>)

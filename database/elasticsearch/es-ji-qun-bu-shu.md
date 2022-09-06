---
description: 10分钟，从0到实现es的集群部署，这里es的版本为7.2.0
---

# ES集群部署

[JDK安装部分](../../shu-ju-ku/elasticsearch/broken-reference/)

先 看一下集群部署可能会遇见的问题：

1. 配置文件正确但是，查看es的节点信息却没有正确显示？

答：可能是因为防火墙的开始导致节点之间无法正常通信，需关闭防火墙。

　2.启动报错，不能以root用户登录？

答：方法１更改elasticsearch文件夹的属主。

```
chown -R elasticsearch:elasticsearch elasticsearch
```

　　如果修改之后依旧启动报错不能以root用户登录，方法2更改/etc/passwd。

![/etc/passwd](<../../.gitbook/assets/image (39).png>)

```
# 修改为
elasticsearch:x:995:991:elasticsearch user:/nonexistent:/bin/bash
```

## 1.安装RPM

　　上传RPM包进行安装

```
# 安装指令
rpm -ivh elasticsearch&.rpm

# 查看是否安装过elasticsearch
rpm -qa | grep elasticsearch

# 卸载elasticsearch服务
rpm -e elasticsearch
```

## 2.修改配置

```
# 修改elasticsearch.yml配置文件 节点名字分别配置为node-1,node-2,node-3
node.name: node-1

# 集群名字需配置一样(三台服务器保持一致)
cluster.name: my_cluster

# 其它地址可以访问(三台服务器保持一致)
# network.host: 0.0.0.0

# 集群通信端口(三台服务器保持一致)
http.port: 9200
transport.tcp.port: 9300

# 集群IP配置(三台服务器保持一致)
discovery.seed_hosts: ["192.168.110.133:9300", "192.168.110.134:9300","192.168.110.135:9300"]

# 节点名称配置(三台服务器保持一致)
cluster.initial_master_nodes: ["node-1","node-2","node-3"]
```

## 3.重启ES

```
# 确保防火墙关闭，或者放行9200,9300端口
# 查看状态
systemctl status elasticsearch

# 启动服务
systemctl start elasticsearch

# 停止服务
systemctl stop elasticsearch
```

## 4.验证节点

```
# 查看当前节点数
curl -XGET http://localhost:9200/_cat/nodes?v

# 查看集群状态
```

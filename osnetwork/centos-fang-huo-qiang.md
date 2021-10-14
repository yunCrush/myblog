---
description: 防火墙的常用命令介绍，系统：centos7
---

# Centos防火墙

### 防火墙

**系统centos7**

Centos7默认的防火墙不是iptables而是firewall

查看防火墙状态

```
firewall-cmd --state
```

开启/关闭防火墙

```
systemctl start firewalld

systemctl stop firewalld
```

禁止开机启动

```
systemctl disable firewalld.service
```

查看开放的端口

```
firewall-cmd --list-all
```

监听端口

```
netstat -lnp | grep 8080
```

开放/关闭端口

```
# 开放端口--permanent重启不会丢失
firewall-cmd --add-port=80/tcp --permanent   
   
firewall-cmd --remove-port=9999/tcp --permanent 
```

重启防火墙

```
# 开放，关闭端口都需要重启防火墙
firewall-cmd --reload 
```

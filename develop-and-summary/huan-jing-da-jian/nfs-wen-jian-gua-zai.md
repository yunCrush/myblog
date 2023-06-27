---
description: 利用docker容器，NFS（network file system）实现文件挂载。
---

# NFS文件挂载

　 提前准备好４个虚拟机，一个用作Server端，其它用作Client端。文件挂载就是将一个文件在Server端设置成一个共享文件夹，通过rpc实时的实现Client端进行文件夹内容的同步。目前得知，在es的集群数据还原时需要进行文件挂载。所以这里创建的共享文件夹也是为ES集群数据迁移做铺垫。关于ES集群数据迁移可以利用左上角进行搜索。

　Server ip: 192.168.110.136

　Client ip: 192.168.110.135, 192.168.110.134, 192.168.110.133

## 1. 服务端配置

**下载启动服务**

```
# 安装nfs-utils rpcbind服务
yum install nfs-utils rpcbind

# 设置开机启动
chkconfig nfs on
chkconfig rpcbind on

# 启动服务 查看状态start => status
service rpcbind start
service nfs start

# 关掉防火墙
systemctl stop firewalld
```

**创建挂载点**

```
# 创建挂载点
mkdir -p /home/crush/esback
```

**挂载目录**

```
# 挂载目录 将前者esback挂载到后者的esback,前者esback作为几个服务器之间同步数据的文件地址
mount -t nfs server_ip:/home/crush/esback  /home/crush/esback
mount -t nfs 192.168.110.136:/home/crush/esback /home/crush/esback
```

**编辑配置文件**

```
# 编辑/etc/exports文件
vim /etc/exports
# 输入以下命令 no_root_squash：来访的root用户保持root帐号权限；
# no_subtree_check ：即使输出目录是一个子目录，nfs服务器也不检查其父目录的权限，这样可以提高效率；
/home/crush/esback client_ip(rw,sync,no_root_squash,no_all_squash)
```

![/etc/exports配置文件](<../../.gitbook/assets/image (2) (1).png>)

**刷新配置文件：`exportfs -a`**

**查看共享目录**

```
showmount -e Server-ip
```

## 2.客户端配置

**客户端同上需要进行基本服务安装与创建挂载点，完成这两步后再进行目录挂载。**

客户端这里配置挂载的目录，同样为/home/crush/esback,三个Client都需要创建，这里同样**关闭防火墙**。

这里用/home/crush/esback，方便更好的理解，表示将server端的文件，挂载到本地的某个文件中

**配置文件修改**

```
# 修改/etc/fstab
vim /etc/fstab

192.168.110.136:/home/crush/esback /home/crush/esback nfs defaults 0 0
```

**目录挂载**

```
# 目录挂载 将Server端 挂载到本地
mount -t nfs server_ip:/home/crush/esback /home/crush/esback
mount -t nfs 192.168.110.136:/home/crush/esback /home/crush/esback
# 查看挂载目录
df -h
```

![](<../../.gitbook/assets/image (3) (1).png>)

测试：在Server端创建文件，会共享到3个Client端，在ES集群数据迁移会有使用到。

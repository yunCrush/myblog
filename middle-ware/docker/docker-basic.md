---
description: docker的安装与常用命令介绍
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

# Docker安装部署

## 1.安装、启动docker

```
# 安装docker
sudo yum -y update

sudo yum -y install epel-release

sudo yum -y install docker-io

# 启动docker
systemclt start docker                           

docker  version                            
```

## 2.docker的常用命令

```
# 查看docker运行的容器
docker ps
                 
# 显示所有容器,-q:quiet静默的标号，即只显示id
docker ps -a
docker ps -qa

# 查看本地镜像
docker images      

# 强制删除指定id的image
docker rmi <image id> 

# 删除所有images
docker rmi $(docker iamges -q)         

# 停止指定id的容器
docker stop <container-id>

# 停止所有的容器
docker stop $(docker ps -aq)

# 移除该id的容器
docker rm <container-id> 
```

## 3.docker启动容器

```
# docker search centos 查看相关镜像
# 拉取centos:7镜像
docker pull centos:7

# -i 交互式 -t 伪终端 -d后台启动 --help查看帮助 run通过镜像来创建容器，--name为容器命名
docker run -it --name centos1 centos:7

# 暂时退出容器，容器保持运行 exit关闭容器
ctrl+p+q

# 启动容器
docker start <container-id>

# 重新进入容器，前提是容器在运行
docker attach <container-id>
docker exec -it <container-names>  /bin/bash
docker exec -it centos1 /bin/bash

# 将容器文件保存到宿主机
docker cp <container-id>:/home/crush/READ.md  /home/crush/

# 宿主机文件保存到容器
docker cp /home/crush/first.md  <container-id>:/home/crush/

# 查看容器日志 -f follow
docker logs <container-id> --tail 5 -f 

# 查看容器进程信息
docker top <container-id>
```

这里只是简单的介绍下命令，后续部署es集群将能更好的进行实践。

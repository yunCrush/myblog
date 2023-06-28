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

# Redis环境篇



```
// yum install -y gcc-c++
mkdir -p /data/soft/
cd /data/soft
wget http://download.redis.io/releases/redis-3.2.3.tar.gz
tar -zxvf redis-3.2.3.tar.gz
cd redis-3.2.3
// 编译
make
cd src/
// 安装将redis-cli安装在usr/local/redis下，将redis.conf配置在同一目录下即可
make install PREFIX=/usr/local/redis
//启动与关闭
./redis-server redis.conf    ./redis-cli shutdown 
//连接客户端的启动
./redis-cli 
```

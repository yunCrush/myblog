# ZooKeeper安装

### 安装部署

```
tar -zxvf apache-zookeeper-3.5.7-bin.tar.gz
# 首先将zoo_sample.cfg重命名为zoo.cfg
#然后修改zoo.cfg中的dataDir参数的值，dataDir指向的目录存储的是zookeeper的核心数据，
#所以这个目录不能使用tmp目录

cd apache-zookeeper-3.5.7-bin/conf/
 mv zoo_sample.cfg zoo.cfg

vim /zoo.cfg
#修改：
dataDir=/data/soft/apache-zookeeper-3.5.7-bin/data

# 启动start, 状态status, 停止stop
bin/zkServer.sh start

#客户端连接工具
bin/zkCli.sh

# 验证,如果能看到QuorumPeerMain进程就说明zookeeper启动成功
jps
```

### 集群部署

```
#编辑配置文件
vim zoo.cfg
dataDir=/data/soft/apache-zookeeper-3.5.8-bin/data
server.0=bigdata01:2888:3888
server.1=bigdata02:2888:3888
server.2=bigdata03:2888:3888


创建目录保存myid文件，并且向myid文件中写入内容
myid中的值其实是和zoo.cfg中server后面指定的编号是一一对应的
编号0对应的是bigdata01这台机器，所以在这里指定0

cd /data/soft/apache-zookeeper-3.5.7-bin/data
echo 0 > myid

# 修改bigdata02,bigdata03的配置文件 myid分别为1,2。
```

### 常用操作

```
# 创建节点，节点即目录，存放数据
create /test hello
# 获取节点
get /test
# 删除节点
deleteall /test
```

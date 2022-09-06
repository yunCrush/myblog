# Kafka安装

### 单机部署

```
cd /data/soft/
tar -zxvf kafka_2.11-2.4.0.tgz 

#broker.id：集群节点id编号，单机模式不用修改
#listeners：默认监听9092端口
#log.dirs：注意：这个目录不是存储日志的，是存储Kafka中核心数据的目录，
#这个目录默认是指向的tmp目录，所以建议修改一下
#zookeeper.connect：kafka依赖的zookeeper

# 编辑配置文件
cd kafka_2.11-2.4.0/config
vi server.properties
log.dirs=/data/soft/kafka_2.11-2.4.0/kafka-logs

# 启动kafka
bin/kafka-server-start.sh -daemon config/server.properties 
# 验证 
jps
# 停止
bin/kafka-server-stop.sh 
```

### 集群部署

```
#此时针对集群模式需要修改 broker.id、log.dirs、以及zookeeper.connect
#broker.id的值默认是从0开始的，集群中所有节点的broker.id从0开始递增即可
#所以bigdata01节点的broker.id值为0
#log.dirs的值建议指定到一块存储空间比较大的磁盘上面，因为在实际工作中kafka中会存储很多数据
#zookeeper.connect的值是zookeeper集群的地址，
#可以指定集群中的一个节点或者多个节点地址，多个节点地址之间使用逗号隔开即可

vi server.properties

broker.id=0
log.dirs=/data/kafka-logs
zookeeper.connect=bigdata01:2181,bigdata02:2181,bigdata03:2181

# 将修改好配置的kafka安装包拷贝到其它两个节点,修改broker.id的值
```

### 常用操作

```
# 想要添加数据首先需要创建topic
# 新增Topic：指定2个分区，2个副本，注意：副本数不能大于集群中Broker的数量
#因为每个partition的副本必须保存在不同的broker，，
#如果你们用的是单机kafka的话，这里的副本数就只能设置为1了，这个需要注意一下
#否则没有意义，如果partition的副本都保存在同一个broker，那么这个broker挂了，
#则partition数据依然会丢失
#在这里我使用的是3个节点的kafka集群
#所以副本数我就暂时设置为2，最大可以设置为3

bin/kafka-topics.sh --create --zookeeper localhost:2181  --partitions 2 --replication-factor 2 --topic hello

#查看所有topic
bin/kafka-topics.sh --list --zookeeper localhost:2181

#查看指定topic
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic hello

# Leader：Leader partition所在的节点编号，这个编号其实就是broker.id的值

# 修改Topic：修改Topic的partition数量，只能增加
bin/kafka-topics.sh --alter --zookeeper localhost:2181 --partitions 5 --topic hello

# 删除Topic：删除Kafka中的指定Topic
bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic hello
```

1. 生产者和消费者

```
# 先创建一个topic【5个分区，2个副本】：
bin/kafka-topics.sh --create --zookeeper localhost:2181  --partitions 5 --replication-factor 2 --topic hello 

#向这个topic中生产数据
#broker-list：kafka的服务地址[多个用逗号隔开]、
# topic：topic名称

bin/kafka-console-producer.sh --broker-list localhost:9092 --topic hello

#创建一个消费者消费topic中的数据,默认消费最新的数据，从头消费：--from-beginning
#bootstrap-server：kafka的服务地址
#topic:具体的topic

# 消费最新
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic hello
# 从头消费
 bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic hello --from-beginning

```

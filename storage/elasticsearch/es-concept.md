# ElasticSearch概念

## 1.对比Solr

## 2.常见概念

**2.1 节点角色**

分为5个角色：

1. 主节点(master node): 负责节点状态的维护，分片的分配，索引的创建和删除，以及数据的Rebalance。不负责数据的检索工作，负载较低，需要一个稳定的服务器。
2. 数据节点(data node): 负责集群中索引的创建和检索，具体包括数据的检索，搜索和聚合，数据节点属于I/O、CPU、Memory密集型,建议使用SSD加快数据的读取。
3. 提取节点(Ingest node): 执行数据预处理的管道，在索引检索前预处理文档，拦截bulk和Indexq请求,加以转换，然后将文档返回给bulk和index API.如果集群需要复杂的预处理逻辑，则该节点属高负载，建议使用专用服务器。
4. 协调节点(coordinating node): 协调节点能够将请求分发到所有的data node，处理客户端的请求,收到data node 返回的结果，进行合并返回给客户端。
5. 部落节点(tribe node)：在多个集群间充当联合客户端角色，实现跨集群访问，在5.4版本后被废弃，替代方案为cross-cluster Search。

```
# 相关配置,这里仅仅介绍如何配置，不同的节点需要配置在不同服务器
node.master: true
node.data: true
node.ingest: true
```

## 3. 集群选举原理

　采用优化的Bully算法，Bully算法是假定每个node都有一个id，对id进行排序，选取最大的id作为候选节点。es在具体实现的是选取最小的id，作为候选节点，es做分布式集群时3个节点以上最好，且为奇数。

　es集群的选举原则：

1. 每个节点像已知的最小的id的节点投票，可像自己投票。
2. es集群中所有节点都会参与选举和投票，只有master角色的节点投票有效。
3. 如果一个节点获得足够多的票数，并且也为自己投票，则该节点将成为Leader角色，开始发布集群状态。

**脑裂问题**

　脑裂问题的产生是由于网络分区导致的，一个集群被划分为两个或者多个集群不能相互通信，最终出现系统混乱，服务异常和数据不一致，为了避免这种脑裂问题es通过如下配置解决：`discover.zen.minimum_master_nodes = (master_eligible_nodes)/2 + 1`

后者表示大于半数的节点个数，在对集群选主的时，会做以下判断：

1. 触发选主：选举临时master是否达到一定数量的节点,即大于3的可用的master角色节点
2. 决定master：选出master后需要票数达到一定的数量，才确认为master节点
3. Gateway 选举元信息：向有Master资格的节点发起请求，获取元数据，获取响应的数量需达到元信息的选举节点数。
4. Master发布集群状态：成功向节点发布集群状态信息的数量需要达到法定数量即选举节点数。
5. NodesFaultDetection: 判断是否触发rejoin，即有节点无法通信时，会执行remove node，此时会判断法定数量是否会大于`discover.zen.minimum_master_nodes`,如果不大于则放弃Master执行rejoin以避免脑裂。

## 4.节点与分片

　green表示节点分片与副本都正常，yellow表示有副本不可用，会影响高可用，red表示节点至少有一个主分片异常，此时检索可正常，但是会存在不返回数据或者返回部分数据。

　集群扩容水平扩容加入新节点时，会做relocation 使得各个节点均衡分布数据

　**分片的容量设置**

　每个分片的文档数最大可达Integer.MAX\_VALUE-128个，官方推荐20-40G,不超过50G。**这里提一下：如果存储es数据的磁盘使用率超过95%,ES索引会变为只读状态**。

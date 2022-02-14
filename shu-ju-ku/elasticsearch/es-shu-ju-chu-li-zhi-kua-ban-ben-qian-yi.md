---
description: ES不同版本之间备份的数据是不能进行使用快照还原的。
---

# ES数据处理之跨版本迁移

　本文中由7.4.0迁移到7.4.2。使用到的是 :point\_right: [elasticdump](https://github.com/elasticsearch-dump/elasticsearch-dump)插件。适用场景：适合数据量不大，迁移索引个数不多的场景，这里测试的数据是在每个索引大约15W的数据量。

## 1 . 安装elasticsearch-dump

```
npm install elasticdump -g
```

## 2 . 参数说明

```
--input: 源地址，可为ES集群URL、文件或stdin,可指定索引，格式为：{protocol}://{host}:{port}/{index}
-index: 源ES集群中的索引
--output: 目标地址，可为ES集群地址URL、文件或stdout，可指定索引，格式为：{protocol}://{host}:{port}/{index}
--output-index: 目标ES集群的索引
--type: 迁移类型，默认为data,表明只迁移数据，可选settings, analyzer, data, mapping, alias
--limit：每次向目标ES集群写入数据的条数，不可设置的过大，以免bulk队列写
```

## 3.迁移单个索引

　　以下操作通过elasticdump命令将集群172.16.0.39中的companydatabase索引迁移至集群172.16.0.20。注意第一条命令先将索引的settings先迁移，如果直接迁移mapping或者data将失去原有集群中索引的配置信息如分片数量和副本数量等。

```
elasticdump --input=http://172.16.0.39:9200/companydatabase --output=http://172.16.0.20:9200/companydatabase --type=settings
elasticdump --input=http://172.16.0.39:9200/companydatabase --output=http://172.16.0.20:9200/companydatabase --type=mapping
elasticdump --input=http://172.16.0.39:9200/companydatabase --output=http://172.16.0.20:9200/companydatabase --type=data
```

　　当然也可以直接在目标集群中将索引创建完毕后再同步mapping与data。

## 4.迁移所有索引

　　以下操作通过elasticdump命令将将集群172.16.0.39中的所有索引迁移至集群172.16.0.20。 注意此操作并不能迁移索引的配置如分片数量和副本数量，必须对每个索引单独进行配置的迁移，或者直接在目标集群中将索引创建完毕后再迁移数据。

```
elasticdump --input=http://172.16.0.39:9200 --output=http://172.16.0.20:9200
```


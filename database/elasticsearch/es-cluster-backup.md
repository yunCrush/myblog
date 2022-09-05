---
description: 这里介绍ES的集群备份。
---

# ES数据处理之集群备份

ES集群备份的前提是相同版本的ES，如果不相同，请参照 :point\_right:[here](es-shu-ju-chu-li-zhi-kua-ban-ben-qian-yi.md)。

## 1.注册备份仓库

**第一部分步骤需在两台服务器都进行操作。**

```
cd /home/crush
mkdir esback
# 为了避免权限问题，直接修改/home/crush/esback权限为777
chmod 777 /home/crush/esback
```

修改配置文件

```
# elasticsearch.yml
path.repo: ['/home/crush/esback']
```

这里的my\_backup与location的位置对应，可以理解为别名，也与ES配置文件的地址对应。

```
# 注册仓库地址方法1
PUT http://地址:9200/_snapshot/my_backup/
# 参数
{
    "type": "fs",
    "settings": {
        "location": "/home/crush/esback",
        "max_snapshot_bytes_per_sec" : "50mb",
        "max_restore_bytes_per_sec" : "50mb",
        "compress" : true
    }
}

# 方法2使用curl 进行注册仓库地址
curl -XPUT http://server_ip:9200/_snapshot/my_backup/ -H 'Content-Type:application/json' -d '{"type":"fs","settings":{"location":"/home/crush/esback","max_snapshot_bytes_per_sec" : "50mb","max_restore_bytes_per_sec" : "50mb","compress" : true}}'
```

## 2.备份数据

**紧在需要备份的服务器进行备份**

（在my\_backup仓库下备份数据，备份的名字为snapshot\_20210407）

```
PUT http://地址:9200/_snapshot/my_backup/snapshot_20210407

# 指定备份的索引逗号分隔，不传默认备份所有索引
{
    "indices": "index1,index2"
}
# 等待备份完
?wait_for_completion=true 
```

备份完数据后就会在/opt/es\_bak目录下生成备份的元数据

**这里介绍一下关于查看快照的相关命令**

```
# kibana运行 查看目前的所有快照
GET _snapshot/

# 查看备份进度 snapshotName对应仓库地址与快照名字
GET _snapshot/snapshotName/_status
```

## 3.单节点与集群区别

集群备份，需要进行 :point\_right:[ NFS文件挂载](../../huan-jing-bu-shu/huan-jing-da-jian/nfs-wen-jian-gua-zai.md)，单节点可直接进行以下恢复操作。

将备份的原始数据打包，可通过[scp文件传输命令](../../Develop\&Summary/chang-yong-ming-ling/linux-basic-command.md#2-wen-jian-de-chuan-shu)传送到，需要恢复的服务器，然后进行数据恢复。

```
POST http://地址:9200/_snapshot/my_backup/snapshot_20210407/_restore

# kibana命令恢复
POST _snapshot/my_backup/snapshot_20210407/_restore
```

## 方案二：使用\_reindex进行迁移

场景：从A服务器迁移索引index1到B服务器

1. 在B服务器ES配置中添加以下配置，并重启服务器

```
reindex.remote.whitelist: A_ip
```

2\. 在B服务器端执行迁移命令,前提是在服务器B的es中需要创建好index1索引的Mapping.

```
POST _reindex
{
    "source": {
        "remote": {
            "host": "http://A_ip:9200"
            },
        "index": "index1"
    },
    "dest": {
        "index": "index1"
    }
}
```

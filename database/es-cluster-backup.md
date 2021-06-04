---
description: 这里介绍ES的集群备份。
---

# ES数据处理之集群备份

ES集群备份的前提是相同版本的ES，如果不相同，请参照 👉[here](es-shu-ju-chu-li-zhi-kua-ban-ben-qian-yi.md)。

## 1.注册备份仓库

**第一部分步骤需在两台服务器都进行操作。**

```text
cd /home/crush
mkdir esback
# 为了避免权限问题，直接修改/home/crush/esback权限为777
chmod 777 /home/crush/esback
```

修改配置文件

```text
# elasticsearch.yml
path.repo: ['/home/crush/esback']
```

这里的my\_backup与location的位置对应，可以理解为别名，也与ES配置文件的地址对应。

```text
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

```text
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

```text
# kibana运行 查看目前的所有快照
GET _snapshot/

# 查看备份进度 snapshotName对应仓库地址与快照名字
GET _snapshot/snapshotName/_status
```

## 3.单节点与集群区别

集群备份，需要进行 👉[ NFS文件挂载](../osnetwork/nfs-wen-jian-gua-zai.md)，单节点可直接进行以下恢复操作。

将备份的原始数据打包，可通过[scp文件传输命令](../osnetwork/linux-basic-command.md#2-wen-jian-de-chuan-shu)传送到，需要恢复的服务器，然后进行数据恢复。

```text
POST http://地址:9200/_snapshot/my_backup/snapshot_20210407/_restore

# kibana命令恢复
POST _snapshot/my_backup/snapshot_20210407/_restore

```


---
description: 这里介绍关于ES的字段名称的修改。
---

# ES属性处理之字段重命名

## 0.需求介绍

```
# 拥有的索引名称与字段
{
    "_index": "testindex",
    "_type": "logs",
    "_id": "1",
    "_score": 1,
    "_source": {
      "field1": "data1",
      "field2": "data2"
    }
}
```

```
# 需要更改后的名称与字段，将field2更改为Request.field3
{
    "_index": "testindex",
    "_type": "logs",
    "_id": "1",
    "_score": 1,
    "_source": {
      "field1": "data1",
      "Request": {
        "field3": "data2"
      }
    }
}
```

## 1.备份索引

```
# 将testindex备份为testindex1，这里的testindex1于临时变量，中间桥梁
POST _reindex
{
    "source": {
        "index": "testindex"
    },
    "dest": {
        "index": "testindex1"
    }
}
```

## 2.删除原索引

```
DELETE testindex
```

## 3.修改名称并恢复索引

```
# 这里的Request字段需要创建，否则会报错
POST _reindex
{
    "source": {
        "index": "testindex1"
    },
    "dest": {
        "index": "testindex"
    },
    "script": {
        "inline": "ctx._source.Request = [field3: ctx._source.remove(\"field2\") ]"
    }
}

# 删除临时索引testindex1
DELETE testindex1
```


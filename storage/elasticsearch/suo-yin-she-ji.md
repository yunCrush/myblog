# 索引设计

优雅的索引设计,决定了存储的成本,主要是从场景来选择对应的数据类型,以及是否需要对当前字段进行索引来设置合适的参数.

这里直接贴代码,仅对个别重点解释,字段并无实际意义,只是为了演示对应数据类型.

```java
PUT yc-index/
{
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword"
      },
      "comments": {
        "type": "nested",
        "properties": {
          "name": {
            "type": "text"
          },
          "age": {
            "type": "integer"
          }
        }
      },
      "download_time": {
        "type": "date",
        "format": "[yyyy-MM-dd HH:mm:ss]"
      },
      "CDN": {
        "type":"boolean"
      },
      
      "request_info": {
        "properties": {
          "category": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "time_from": {
            "type": "date",
            "format": "[yyyy-MM-dd]"
          },
          "time_to": {
            "type": "date",
            "format": "[yyyy-MM-dd]"
          },
          "type": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      },
      "task_id": {
        "type": "keyword"
      },
      "task_flag": {
        "type": "integer",
        "index": false,
        "doc_values": false
      },
      "username": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      }
    }
  }
}

```

`index: false`：这个参数指定了是否在索引中存储字段的原始值。当你将一个字段设置为 index: false，意味着这个字段不会被用于搜索。它不会被分析，也不会被建立倒排索引，因此在搜索时无法通过这个字段进行匹配。这个参数通常用于存储一些不需要进行搜索的原始数据，以减少索引的大小和提高查询性能

`doc_values: false`：这个参数控制是否在磁盘上存储字段的排序和聚合数据。默认情况下，Elasticsearch会为每个字段建立倒排索引以支持快速的全文搜索和匹配。然而，当你对一个字段设置 `doc_values: false`，Elasticsearch不会为该字段建立这样的数据结构，因此无法使用该字段进行排序、聚合和使用脚本进行计算。这个参数通常用于一些不需要排序或聚合操作的字段，以节省磁盘空间

```java

"task_flag": {
        "type": "integer",
        "index": "false",
        "doc_values": "false"
      }
      
 "comments": {
 		# comments属性有多个name-age构成的对象,用作父子文档关联
        "type": "nested",
        "properties": {
          "name": {
            "type": "text"
          },
          "age": {
            "type": "integer"
          }
        }
      }
```

针对"nested"类型进行搜索时需要指定path.

```java
GET yc-index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "nested": {
            "path": "comments",
            "query": {
              "bool": {
                "must": [
                  {
                    "match": {
                      "comments.name": "john"
                    }
                  },
                  {
                    "match": {
                      "comments.age": 34
                    }
                  }
                ]
              }
            }
          }
        }
      ]
    }
  }
}

```

join关键字同nested用法相同,用作父子文档关联

```

    # rel代表一个字段
    "rel" : {
          "type" : "join",
          # 开启全局序列号便于聚合
          "eager_global_ordinals" : true,
          "relations" : {
          # org -> event 父子关系
            "org" : "event",
            # event -> relationship 父子关系
            "event" : "relationship"
          }
        },
```

查询出的结果是这样的

```
"rel" : {
			# 可以看出当前数据的parent是bundle...
            "parent" : "bundle--e74df260-ff09-3598-ba95-8cfdd81b065b",
            # 当前的数据的类型是relationship
            "name" : "relationship"
          }
```

总结: 对于不同的场景使用不同的数据类型,不建议使用nested和join类型,在大数据量情况下查询影响性能.

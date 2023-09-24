# ES深度分页方案

常见的三种ES查询方案

### From+Size

正常查询，每个分片查询from+size个文档，协调节点再进行相关度算分，排序后再返回size个节点给客户端，涉及到深度分页问题

### Scroll

scroll 生成一个快照，每次查询返回一个id,通过这个id一直查询，底层有一个游标在移动，直到查不出来为止，因此如果有新写入的数据，是无法查询出来的

### SearchAfter

searchAfter 不支持指定页数(from),只能往下翻，第一步搜索时需要指定sort,并且保证值是唯一的(可以通过加入\_id保证唯一性) 然后使用上一次最后一个文档的sort值进行查询

![searchAfter-1](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/ES-SearchAfter-1.png)

![searchAfter-2](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/ES-SearchAfter-2png)

![searchAfter-3](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/ES-SearchAfter-3.png)

SearchAfter是通过在每个分片上面返回size个文档，解决深度分页问题的。正常查询的话是需要返回from+size个文档，需要全部导出就用Scroll

---
description: 记录在使用ES java API开发过程中遇见的坑，以及解决方案。
---

# ES搜索踩坑日记

ES Java API开发大体思路：

```text
# 构建以下请求对象
SearchRequest request = new SearchRequest(indexName);
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
searchSourceBuilder.query(queryBody);
request.source(searechSourceBuilder);

# 发起请求与响应，发送请求会抛异常，应将异常抛至顶层
SearchResponse response = esClient.search(request,RequestOptions.DEFAULT);
if(response.hasFailures()){
    ....处理逻辑
}

# 解析返回数据，业务逻辑处理
SearchHits hits = searchResponse.getHits();

# 命中文档总数
long total = hits.getTotalHits().value;

SearchHit [] searchHits = hits.getHits();
Map<> searchResultMap = new HashMap<>();
for(SearchHit hit : searchHits){
    searchResultMap = hit.getSourceAsMap();
# 通过searchResultMap.get()获取数据库中字段即可，注意类型转换
}
```

## 问题1：分页问题，聚合后不能使用分页

```text
# 每页size,当前第current页
searchSourceBuilder.from((current-1)*size);
searchSourceBuilder.size(size)
```

## 问题2: ES搜索返回数据数量不全

A.常见的命中所有文档数量超过1W，默认只返回1W。

B.简单查询，默认只返回10条数据

```text
# 将所有命中数据文档全部返回，解决问题A
searchSourceBuilder.trackTotalHits(true);

# 解决问题B两种方法，一种同上
# 另一种设置一个远大于数据总量的值，不建议，假如我们知道数据最多10+，size设置200
searchSourceBuilder.size(size);
```

## 问题3：数据不及时同步更新

```text
# 通过JAVA API删除数据后，通过API查询数据仍然存在，表示删除的数据没有及时的刷新
# 这里根据业务需要设置刷新的参数
UpdateRequest request;
request.setRefreshPolicy(IMMEDIATE)
```

## 问题4：注意数据的格式问题

```text
# 数据格式问题不对，注意数据字段是keyword，还是text下的keyword，会导致解析"_source"时报错
```

## 问题5：根据某个字段进行排序

```text
# String fieldName 根据fieldName升序ASC 降序DESC
searchSourceBuilder.sort(new FieldSortBuilder(fieldName)).order(Sortder.ASC));
```

## 问题6：模糊查询与全匹配

```text
# must即"与 &&"操作
BoolQueryBuilder boolQueryBuilder  = new BoolQueryBuilder();

# 匹配name字段的值是name的，全匹配
boolQueryBuilder.must(QueryBuilders.matchQuery("name",name);

# 模糊查询
boolQueryBuilder.must(QueryBuilders.wildcardQuery("name", "*"+name+"*"));
```

## 问题7：返回文档中的某个字段的所有值

```text
# 插入数据
PUT items/1
{ "language" : 10 }

PUT items/2
{ "language" : 11 }

PUT items/3
{ "language" : 10 }

# 我们期望的结果是这样的,返回10，11
GET items/_search
{ ... }

[10, 11]

# 方法1
GET items/_search
{
"size": 0,
"aggs" : {
    "langs" : {
        "terms" : { "field" : "language",  "size" : 500 }
            }
      }
}

# 方法2
GET items/_search
{
"_source": "language"
}
```

参考来源 👉 [ here](https://stackoverflow.com/questions/25465215/elasticsearch-return-unique-values)

**JAVA API**

```text
# 传入的idList：1，2，3， includes 查询的字段， excludes不包括的字段
List<String> list = new ArrayList<>();
MultiGetRequest multiGetRequest = new MultiGetRequest();
String [] includes = {"language"}
String [] excludes = String.EMPTY_ARRAY;
FetchSourceContext context = new FetchSourceContext(true,includes,excludes);

# esIndexName 索引名字
for(String id : idList){
 multiGetRequest.add(new MultiGetRequest.Item(esIndexName,id)
 .fetchSourceContext(context));
}

# 获取响应,注意异常处理,esClient为es客户端实例
MultiGetResponse response = esClient.mget(multiGetRequest,RequestOptions.DEFAULT);
MultiGetItemResponse [] itemResponse = response.getResponses();
for(MultiGetItemResponse item : itemResponse){
   if(item.getFailure() != null){
   continue;
   }
   GetResponse getResponse = item.getResponse();
   if(getResponse.isExists()){
      Map<String,Object> sourceMap = getResponse().getSourceAsMap();
      list.add(sourceMap.get(field).toString);
   }

}
return list;
```

## 问题8：curl命令

```text
# 初始化mapping
curl -XPUT 'http://localhost:9200/index-name' -d @file-path/index-name.json --header "Content-Type:application/json"
```

## 问题9：返回数据"\_source"下某个字段去重

```text
# 使用collapse进行去重,此方法仅仅对keyword类型与number有效
GET my_index/_search
{
    "query":{
        "match":{
            "name": "yuncrush"
        }
    },
     "collapse":{
        "field":  "name.keyword"
    }
}
```

## 问题10：聚合操作

```text
# 背景：两个字段soure_name表示来源名字，check_time表示检查时间
# 需要通过来源名字进行聚合，然后筛选出check_time是最新时间的一条数据
# group_aggs time_aggs 表示分组后桶的名字

AggregationBuilder groupAggBuilder = AggregationBuilders.terms("group_aggs").
field("source_name").size(10000);

AggregationBuilder timeAggBuilder = AggregationBuilders.topHits("time_aggs").
sort("check_time",SortOrder.DESC).size(1);

groupAggBuilder.subAggregation(timeAggBuilder);
searchSourceBuilder.aggregation(groupAggBuilder);
searchSourceBuilder.timeout(new TimeValue(300));
searchSourceBuilder.trackTotalHits(true);
request.source(searchSourceBuilder);

SearchResponse response = esClient.search(request,RequestOptions.DEFAULT);
Terms gropuAggs = response.getAggregations().get("group_aggs");

if(Objects.nonNull(groupAggs)){
    for(Terms.Bucket groupBucket : groupAggs.getBuckets()){
    ......字段处理
    TopHits topHits = groupBucket.getAggregations().get("time_aggs");
    SearchHit[] searchHit = topHits.getHits.getHits();
    .....字段处理

}
```




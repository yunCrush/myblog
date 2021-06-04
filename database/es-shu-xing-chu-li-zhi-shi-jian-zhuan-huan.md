---
description: 这里介绍一种ES的Date时间格式转换的方法。
---

# ES属性处理之时间转换

背景：有这样一个字段“expired\_date”，查看mapping存放的类型是这样的:

```text
"expired_date":{
    "type": "date",
    "format": "yyyy-MM-dd HH:mm:ss"
}
```

ES数据库中存放的数据是这样的：“2022-01-01T:00:00:00”

在进行数据备份的时候，希望存放的“expired\_date”的格式是这样的：“2022-01-01 00:00:00”。起初并没有注意到时间存放的数据含有T，在进行备份的时候出现转换异常，才知道需要修改时间格式。

**解决办法**

```text
# 使用_reindex进行数据备份，在备份时，进行时间格式转换
POST _reindex?wait_for_completion=false
{
  "source": {
    "index": "source_index"
  },
  "dest": {
    "index": "dest_index"
  },
  "script": {
    "source": """
                  def sou = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss");
                  def des = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                  if(ctx._source.expired_date != null){
                         ctx._source.expired_date=des.format(sou.parse(ctx._source.expired_date));
                  }
                  else{
                         ctx.source.expired_date=des.format(new Date());
                  }
                  """
  }
}
```

这里额外增加一个知识点，如果遇见数据备份网络超时解决办法，使用如上的`wait_for_completion=false`

这里会返回一个task id, 通过比较两个索引的数据总量可以判断数据是否备份完成。

```text
 {"statusCode":502,"error":"Bad Gateway","message":"Client request timeout"}
```


# 筛选聚合
多桶聚合，每个桶包含匹配查询的文档。

例如：

```json
PUT /logs/_bulk?refresh
{ "index" : { "_id" : 1 } }
{ "body" : "warning: page could not be rendered" }
{ "index" : { "_id" : 2 } }
{ "body" : "authentication error" }
{ "index" : { "_id" : 3 } }
{ "body" : "warning: connection timed out" }

GET logs/_search
{
  "size": 0,
  "aggs" : {
    "messages" : {
      "filters" : {
        "filters" : {
          "errors" :   { "match" : { "body" : "error"   }},
          "warnings" : { "match" : { "body" : "warning" }}
        }
      }
    }
  }
}
```

以上示例，会分析日志信息。该聚合会构建两个桶的日志信息，一个是包含 error 所有文档，另一个是包含 warning 的所有文档。

响应：

```json
{
  "took": 9,
  "timed_out": false,
  "_shards": ...,
  "hits": ...,
  "aggregations": {
    "messages": {
      "buckets": {
        "errors": {
          "doc_count": 1
        },
        "warnings": {
          "doc_count": 2
        }
      }
    }
  }
}
```



## 匿名多筛选
正如以下请求, `filters` 字段也可以设置为筛选数组：

```json
GET logs/_search
{
  "size": 0,
  "aggs" : {
    "messages" : {
      "filters" : {
        "filters" : [
          { "match" : { "body" : "error"   }},
          { "match" : { "body" : "warning" }}
        ]
      }
    }
  }
}
```

在这个请求中，经过筛选的桶会以数组中同样的顺序返回。响应如下：

```json
{
  "took": 4,
  "timed_out": false,
  "_shards": ...,
  "hits": ...,
  "aggregations": {
    "messages": {
      "buckets": [
        {
          "doc_count": 1
        },
        {
          "doc_count": 2
        }
      ]
    }
  }
}
```



## 其他桶
通过设置 `other_bucket` 参数可以添加一个包含所有筛选未匹配文档的桶。参数值如下：

| 合法值  | 说明                                                                       |
|---------|----------------------------------------------------------------------------|
| `false` | 不计算其他桶                                                               |
| `true`  | 如果指定名称（默认 `_other_`）则返回其他桶，若采用匿名多筛选则是最后一个桶 |

通过设置 `other_bucket_key` 参数可以设置其他桶的键值，而不是默认值 `_other_`. 设置这个参数将隐性地设置 `other_bucket` 参数为 `true`.

以下代码片段展示了命名为 `other_messages` 的其他桶的请求方法。

```json
PUT logs/_doc/4?refresh
{
  "body": "info: user Bob logged out"
}

GET logs/_search
{
  "size": 0,
  "aggs" : {
    "messages" : {
      "filters" : {
        "other_bucket_key": "other_messages",
        "filters" : {
          "errors" :   { "match" : { "body" : "error"   }},
          "warnings" : { "match" : { "body" : "warning" }}
        }
      }
    }
  }
}
```

响应如下：

```json
{
  "took": 3,
  "timed_out": false,
  "_shards": ...,
  "hits": ...,
  "aggregations": {
    "messages": {
      "buckets": {
        "errors": {
          "doc_count": 1
        },
        "warnings": {
          "doc_count": 2
        },
        "other_messages": {
          "doc_count": 1
        }
      }
    }
  }
}
```

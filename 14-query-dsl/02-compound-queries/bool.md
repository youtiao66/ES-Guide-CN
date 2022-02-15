# Boolean 布尔查询

布尔查询可以将其他查询组合起来。布尔查询映射 Lucene `BooleanQuery`。它由一个或多个语句构建，每个语句都需要一个类型参数。这些类型参数如下：

| 参数     | 描述                                                                         |
|----------|------------------------------------------------------------------------------|
| must     | 文档**必须**匹配这些条件，对评分有贡献                                       |
| filter   | 文档**必须**匹配这些条件，与`must`不同的是查询积分将被忽略，以过滤模式来执行 |
| should   | 文档匹配这些语句中的**任意**语句                                             |
| must_not | 文档**必须不**匹配这些条件，以过滤模式来执行，查询积分将被忽略               |

布尔查询采取**匹配更多响应更好**的方式，所以来源于每个匹配的`must`或者`should`语句将被添加到每个文档的最终评分。

```js
POST _search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "user.id" : "kimchy" }
      },
      "filter": {
        "term" : { "tags" : "production" }
      },
      "must_not" : {
        "range" : {
          "age" : { "gte" : 10, "lte" : 20 }
        }
      },
      "should" : [
        { "term" : { "tags" : "env1" } },
        { "term" : { "tags" : "deployed" } }
      ],
      "minimum_should_match" : 1,
      "boost" : 1.0
    }
  }
}
```

## `minimum_should_match`
可以使用`minimum_should_match`参数指定`should`语句返回文档必须匹配的数量或百分比

如果布尔查询至少包含一个`should`语句并且没有`must`和`filter`语句，默认值是`1`，否则默认值是`0`

其他可用值，参考[`minimum_should_match`参数][minimum_should_match]

## `bool.filter`评分
在`filter`元素下的查询对评分没有影响——也就是评分返回`0`. 评分仅受指定查询影响。例如，下面三个查询都返回所有`status`字段包含精确值`active`的文档

第一个查询返回的所有文档评分为`0`. 没有指定任何具有评分的查询

```js
GET _search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

以下布尔查询包含`match_all`查询，所有文档评分为`1.0`

```js
GET _search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

`constant_score`查询和上面第二个查询表现基本相同。`constant_score`查询通过筛选匹配向所有文档分配评分`1.0`

```js
GET _search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

## 查询命名
每种查询在顶层接受一个`_name`参数。可以通过该名称追踪查询匹配的文档。如果采用查询命名，每个命中的结果将包含`matched_queries`属性。

```js
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "name.first": { "query": "shay", "_name": "first" } } },
        { "match": { "name.last": { "query": "banon", "_name": "last" } } }
      ],
      "filter": {
        "terms": {
          "name.last": [ "banon", "kimchy" ],
          "_name": "test"
        }
      }
    }
  }
}
```



[minimum_should_match]: [https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-minimum-should-match.html]
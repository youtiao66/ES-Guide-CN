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



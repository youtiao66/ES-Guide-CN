# Match phrase 短语匹配

`match_phrase`查询首先将查询字符串解析成词项列表，然后对这些词项进行短语匹配查询。例如

```js
GET /_search
{
  "query": {
    "match_phrase": {
      "message": "this is a test"
    }
  }
}
```

短语查询根据配置`slop`(默认值0)以任何顺序匹配词项。置换顺序的`slop`为2.

可以配置用于控制文本分析的`analyzer`. 默认值为字段映射中显示定义的分析器或者默认搜索分析器。例如：

```js
GET /_search
{
  "query": {
    "match_phrase": {
      "message": {
        "query": "this is a test",
        "analyzer": "my_analyzer"
      }
    }
  }
}
```

该查询也接受`zero_terms_query`参数，具体参考[匹配查询][match]



[match]: https://youtiao66.gitbook.io/es-guide-cn/query-dsl/full-text-queries/match

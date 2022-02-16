# 搜索结果排序
允许添加一个或者多个指定字段排序。每一种排序都可以逆向。排序通过特殊的字段名称定义在每个字段层级上, `_score`代表通过评分排序, `_doc`代表通过索引排序

假设有以下索引映射：

```json
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "post_date": { "type": "date" },
      "user": {
        "type": "keyword"
      },
      "name": {
        "type": "keyword"
      },
      "age": { "type": "integer" }
    }
  }
}
```

```json
GET /my-index-000001/_search
{
  "sort" : [
    { "post_date" : {"order" : "asc", "format": "strict_date_optional_time_nanos"}},
    "user",
    { "name" : "desc" },
    { "age" : "desc" },
    "_score"
  ],
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

> 💬**说明：** `_doc`除了是最高效的排序以外并没有真正的使用场景。所以如果你不关心文档返回的排序情况下，你需要采用`_doc`排序。这非常有助于 [scrolling][scrolling]



[scrolling]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/paginate-search-results.html#scroll-search-results

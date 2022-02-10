# Terms 多值聚合
基于词项的多值聚合，根据库中的文档动态构建桶，会对一个一个的词项进行聚合

例如：

```js
GET /_search
{
  "aggs": {
    "genres": {
      "terms": { "field": "genre" }
    }
  }
}
```

响应：

```js
{
  ...
  "aggregations": {
    "genres": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "electronic",
          "doc_count": 6
        },
        {
          "key": "rock",
          "doc_count": 3
        },
        {
          "key": "jazz",
          "doc_count": 2
        }
      ]
    }
  }
}
```

1. `doc_count_error_upper_bound` - an upper bound of the error on the document counts for each term. see [below][below]
2. `sum_other_doc_count` - 当存在很多唯一词项时，仅返回排名前列的词项。该数字是响应之外所有桶的文档计数总和
3. `buckets` - 排名靠前的桶列表，通过 [order][order] 定义排名顺序

字段类型可以是 [Keyword][Keyword], [Numeric][Numeric], [ip], [boolean][boolean] 和 [binary][binary]

> 💬**说明：** 默认情况下，多值聚合不能用于`text`字段。可以用`keyword` [sub-field][sub-field] 替代。你也可以通过在文本字段中开启 [fielddata][fielddata] 为字段的解析词项创建桶。开启它可以显著地提高内存使用率

## Size
默认情况下，多值聚合返回前十个最多的文档。可以通过`size`参数返回更多的词项桶，受 [search.max_buckets][search.max_buckets] 限制

如果数据包含 100 或者 1000 个唯一词项，你可以通过增大`size`返回所有。如果有更多的唯一词项并且需要返回所有，使用 [composite aggregation][composite-aggregation] 替代

`size`值越大就会消耗更多的内存计算，并且整个聚合越来越靠近`max_buckets`限制。如果请求失败并返回包含`max_buckets`的错误信息那么就是该参数设置太大了

## Shared Size
分片大小

为了获取更精确的结果，多值聚合从每个分片上获取超过排名靠前`size`的词项。每个分片获取的大小就是`shard_size`，默认值`size * 1.5 + 10`

这是为了兼容以下案例：





[below]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/search-aggregations-bucket-terms-aggregation.html#terms-agg-doc-count-error
[order]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-order
[Keyword]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/keyword.html
[Numeric]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/number.html
[ip]: [https://www.elastic.co/guide/en/elasticsearch/reference/7.15/ip.html]
[boolean]: [https://www.elastic.co/guide/en/elasticsearch/reference/7.15/boolean.html]
[binary]: [https://www.elastic.co/guide/en/elasticsearch/reference/7.15/binary.html]
[sub-field]: [https://www.elastic.co/guide/en/elasticsearch/reference/7.15/multi-fields.html]
[fielddata]: [https://www.elastic.co/guide/en/elasticsearch/reference/7.15/fielddata.html]
[search.max_buckets]: [https://www.elastic.co/guide/en/elasticsearch/reference/7.15/search-settings.html#search-settings-max-buckets]
[composite-aggregation]: [https://www.elastic.co/guide/en/elasticsearch/reference/7.15/search-aggregations-bucket-composite-aggregation.html]

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

这是为了兼容以下案例：当一个词项在第一个分片中有很多文档（排名靠前），但是在其它分片中排名落后于`size`阈值。如果每个分片仅返回排名靠前`size`，聚合就只能够返回词项的一部分文档。所以多值聚合获取更多的词项用于捕获这部分缺失的文档。这能起作用，但仍然返回词项的一部分文档。只要每个分片有完全不同的文档计数就可能会发生该问题。

> **举个例子：**

**我们想要获取name字段中出现频率最高的前5个。**

**此时，客户端向ES发送聚合请求，主节点接收到请求后，会向每个独立的分片发送该请求。**
**分片独立的计算自己分片上的前5个name，然后返回。当所有的分片结果都返回后，在主节点进行结果的合并，再求出频率最高的前5个，返回给客户端。**

**这样就会造成一定的误差，比如最后返回的前5个中，有一个叫A的，有50个文档；B有49。 但是由于每个分片独立的保存信息，信息的分布也是不确定的。 有可能第一个分片中B的信息有2个，但是没有排到前5，所以没有在最后合并的结果中出现。 这就导致B的总数少计算了2，本来可能排到第一位，却排到了A的后面。**

> **本例摘选自[Elasticsearch聚合之Terms][https://www.cnblogs.com/xing901022/p/4947436.html]**

可以通过提高`shard_size`的值更好地考虑这些不同的文档计数并提高多值聚合排名的准确性。提高`shard_size`比提高`size`内存成本更低，但它仍然需要更多的字节通过线路，并在协调节点上的内存中等待。

> ⚠️**重要：** 该指引仅适用于默认排序的多值聚合，如果你不是通过文档计数降序排列的，请参考 [order][order]

> 💬**说明：** `shard_size`的值不能小于`size`（因为这没有什么意义）。如果它的确发生了，`shard_size`将会被覆盖并重置为等于`size`





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

# 筛选聚合
筛选聚合是一个筛选出匹配查询的部分文档的单桶聚合

例如：

```json
POST /sales/_search?size=0&filter_path=aggregations
{
  "aggs": {
    "avg_price": { "avg": { "field": "price" } },
    "t_shirts": {
      "filter": { "term": { "type": "t-shirt" } },
      "aggs": {
        "avg_price": { "avg": { "field": "price" } }
      }
    }
  }
}
```

上面这个例子计算所有销售额平均值，也计算所有 *T恤衫* 销售额平均值

响应：

```json
{
  "aggregations": {
    "avg_price": { "value": 140.71428571428572 },
    "t_shirts": {
      "doc_count": 3,
      "avg_price": { "value": 128.33333333333334 }
    }
  }
}
```



## 使用顶层参数`query`限制所有聚合
通过使用顶层参数`query`限制所有聚合。这比通过子聚合实现单独的筛选聚合更快

例如，推荐如下使用：

```json
POST /sales/_search?size=0&filter_path=aggregations
{
  "query": { "term": { "type": "t-shirt" } },
  "aggs": {
    "avg_price": { "avg": { "field": "price" } }
  }
}
```

而不是：

```json
POST /sales/_search?size=0&filter_path=aggregations
{
  "aggs": {
    "t_shirts": {
      "filter": { "term": { "type": "t-shirt" } },
      "aggs": {
        "avg_price": { "avg": { "field": "price" } }
      }
    }
  }
}
```



## 多个筛选使用多筛选聚合
使用[多筛选聚合][filters-aggregation]使用多个筛选对文档进行分组，这比使用多个筛选聚合更快。

例如，推荐如下使用：

```json
POST /sales/_search?size=0&filter_path=aggregations
{
  "aggs": {
    "f": {
      "filters": {
        "filters": {
          "hats": { "term": { "type": "hat" } },
          "t_shirts": { "term": { "type": "t-shirt" } }
        }
      },
      "aggs": {
        "avg_price": { "avg": { "field": "price" } }
      }
    }
  }
}
```

而不是：

```json
POST /sales/_search?size=0&filter_path=aggregations
{
  "aggs": {
    "hats": {
      "filter": { "term": { "type": "hat" } },
      "aggs": {
        "avg_price": { "avg": { "field": "price" } }
      }
    },
    "t_shirts": {
      "filter": { "term": { "type": "t-shirt" } },
      "aggs": {
        "avg_price": { "avg": { "field": "price" } }
      }
    }
  }
}
```

<!-- TODO: 更改多筛选聚合地址 -->

[filters-aggregation]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/search-aggregations-bucket-filters-aggregation.html

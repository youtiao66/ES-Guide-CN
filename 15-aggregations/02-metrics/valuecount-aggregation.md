# Value count 值计数聚合
单值指标聚合，可以统计出提取自聚合文档的值的数量。这些值可以提取自文档中的指定字段也可以通过提供的脚本生成。通常，这个聚合用于与其他单值指标聚合结合。例如，计算`avg`可能对计算平均数的值数量感兴趣。

值计数聚合不减少重复值。所以即使一个字段有重复值，每个值都将单独计算。

```json
POST /sales/_search?size=0
{
  "aggs" : {
    "types_count" : { "value_count" : { "field" : "type" } }
  }
}
```

响应：

```json
{
  ...
  "aggregations": {
    "types_count": {
      "value": 7
    }
  }
}
```

聚合的名称（上方的`types_count`）还可以用作键，通过该键可以从返回的响应中检索聚合结果。

## 脚本
如果需要计算相对于单个字段更为复杂的情况，可以通过 [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/runtime.html) 实现。

```json
POST /sales/_search
{
  "size": 0,
  "runtime_mappings": {
    "tags": {
      "type": "keyword",
      "script": """
        emit(doc['type'].value);
        if (doc['promoted'].value) {
          emit('hot');
        }
      """
    }
  },
  "aggs": {
    "tags_count": {
      "value_count": {
        "field": "tags"
      }
    }
  }
}
```

## 条形图字段
如果值计数聚合用于计算 [histogram fields](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/histogram.html)

计算结果就是所有条形图计数数组的数字总和

例如，以下索引用于存储已经聚合好不同网络延迟指标的条形图

```json
PUT metrics_index/_doc/1
{
  "network.name" : "net-1",
  "latency_histo" : {
      "values" : [0.1, 0.2, 0.3, 0.4, 0.5],
      "counts" : [3, 7, 23, 12, 6] 
   }
}

PUT metrics_index/_doc/2
{
  "network.name" : "net-2",
  "latency_histo" : {
      "values" :  [0.1, 0.2, 0.3, 0.4, 0.5],
      "counts" : [8, 17, 8, 7, 6] 
   }
}

POST /metrics_index/_search?size=0
{
  "aggs": {
    "total_requests": {
      "value_count": { "field": "latency_histo" }
    }
  }
}
```

每个条形图字段的值计数聚合都将把所有计数数组中的值相加。最终，将对所有条形图的值相加并返回以下结果：

```json
{
  ...
  "aggregations": {
    "total_requests": {
      "value": 97
    }
  }
}
```


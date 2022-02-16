# Sum 总数聚合
单值指标聚合，对提取自聚合文档的数字类型进行求和。这些值可以提取自文档中的指定字段也可以是条形图字段。

假设数据由代表销售记录的文档组成，就可以把所有帽子的销售金额求和

```json
POST /sales/_search?size=0
{
  "query": {
    "constant_score": {
      "filter": {
        "match": { "type": "hat" }
      }
    }
  },
  "aggs": {
    "hat_prices": { "sum": { "field": "price" } }
  }
}
```

响应：

```json
{
  ...
  "aggregations": {
    "hat_prices": {
      "value": 450.0
    }
  }
}
```

聚合的名称（上方的`hat_prices`）还可以用作键，通过该键可以从返回的响应中检索聚合结果。

## 脚本
如果需要计算相对于单个字段更为复杂的情况，可以通过 [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/runtime.html) 实现。

```json
POST /sales/_search?size=0
{
  "runtime_mappings": {
    "price.weighted": {
      "type": "double",
      "script": """
        double price = doc['price'].value;
        if (doc['promoted'].value) {
          price *= 0.8;
        }
        emit(price);
      """
    }
  },
  "query": {
    "constant_score": {
      "filter": {
        "match": { "type": "hat" }
      }
    }
  },
  "aggs": {
    "hat_prices": {
      "sum": {
        "field": "price.weighted"
      }
    }
  }
}
```

## 缺失值
`missing`参数定义了缺失值的文档如何处理。默认被忽略，但也可以被处理为有值。

例如，这里所有缺失销售额的帽子都将被视作`100`

```json
POST /sales/_search?size=0
{
  "query": {
    "constant_score": {
      "filter": {
        "match": { "type": "hat" }
      }
    }
  },
  "aggs": {
    "hat_prices": {
      "sum": {
        "field": "price",
        "missing": 100 
      }
    }
  }
}
```

`tag`字段缺失值的文档将被分入到拥有`N/A`值的同一个桶中

## 条形图字段
如果用于计算 [histogram fields](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/histogram.html)

计算结果就是所有值数组元素乘以计数数组相同位置的元素得到的结果求和

例如，以下索引存储了已经聚合好不同网络延迟指标的条形图

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
  "aggs" : {
    "total_latency" : { "sum" : { "field" : "latency_histo" } }
  }
}
```

对于每个条形图字段，总数聚合都将值数组中的元素和计数数组中与之关联的元素相乘。

最终，将对所有条形图的值相加并返回以下结果：

```json
{
  ...
  "aggregations": {
    "total_latency": {
      "value": 28.8
    }
  }
}
```

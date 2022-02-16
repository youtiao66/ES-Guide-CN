# Cardinality 基数聚合
单值指标聚合，可以计算出不同值的近似计数。

假设你正在为商店销售建立索引，并希望计算与查询匹配的已售产品唯一数量

```js
POST /sales/_search?size=0
{
  "aggs": {
    "type_count": {
      "cardinality": {
        "field": "type"
      }
    }
  }
}
```

响应：

```javascript
{
  ...
  "aggregations": {
    "type_count": {
      "value": 3
    }
  }
}
```

## 精准控制
该聚合还支持`precision_threshold`选项：

```js
POST /sales/_search?size=0
{
  "aggs": {
    "type_count": {
      "cardinality": {
        "field": "type",
        "precision_threshold": 100 
      }
    }
  }
}
```

`precision_threshold`选项允许以内存交换精度，并定义了一个唯一的计数，低于该值时，期望计数将接近于准确。高于该值时，计数可能会变得模糊（误差变大）。最大计数是40000，高于这个阈值跟40000效果一样，默认值是`30000`

## 计数是近似值
TODO:

## 预计算哈希
TODO:

## 脚本
如果需要由两个字段组成基数排序，通过创建 [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/runtime.html) 组合并聚合

```json
POST /sales/_search?size=0
{
  "runtime_mappings": {
    "type_and_promoted": {
      "type": "keyword",
      "script": "emit(doc['type'].value + ' ' + doc['promoted'].value)"
    }
  },
  "aggs": {
    "type_promoted_count": {
      "cardinality": {
        "field": "type_and_promoted"
      }
    }
  }
}
```

## 缺失值
`missing`参数定义了缺失值的文档如何处理。默认被忽略，但也可以被处理为有值。

```json
POST /sales/_search?size=0
{
  "aggs": {
    "tag_cardinality": {
      "cardinality": {
        "field": "tag",
        "missing": "N/A" 
      }
    }
  }
}
```

`tag`字段缺失值的文档将被分入到拥有`N/A`值的同一个桶中


# Cardinality 基数聚合
单指标聚合，可以计算出不同值的近似计数。

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



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



## 排序值
搜索响应包含每个文档排序值。使用`format`参数指定 [date][date] 和 [date_nanos][date_nanos] 字段排序值的 [date format][date-format]. 以下搜索返回以`strict_date_optional_time_nanos`格式表示`post_date`字段的排序值。

```json
GET /my-index-000001/_search
{
  "sort" : [
    { "post_date" : {"format": "strict_date_optional_time_nanos"}}
  ],
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```



## 排序次序
`order`选项有以下值：

| 合法值 | 说明 |
|--------|------|
| `asc`  | 升序 |
| `desc` | 降序 |

`_score`默认排序为`desc`, 其他默认排序为`asc`



## 排序`mode`选项
TODO:

## 排序`mode`使用示例
TODO:

## 数字字段排序
TODO:

## 嵌套对象排序
TODO:

## 嵌套排序示例
TODO:



## 缺失值
`missing`参数定义了缺失值的文档如何处理。

缺失值可以被设置为`_last`, `_first`或者自定义值（该值被用作缺失文档的排序值）。默认为`_last`

例如：

```json
GET /_search
{
  "sort" : [
    { "price" : {"missing" : "_last"} }
  ],
  "query" : {
    "term" : { "product" : "chocolate" }
  }
}
```

> 💬**说明：** 如果嵌套子对象不匹配`nested_filter`将使用缺失值



## 忽略非映射字段
TODO:

## 地理距离排序
TODO:

### 经纬度作为属性

### 经纬度作为字符串

### 经纬度哈希

### 经纬度作为数组

## 多个索引点
TODO:

## 基于脚本排序
TODO:

## 跟踪评分
TODO:

## 内存注意事项
TODO:



[scrolling]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/paginate-search-results.html#scroll-search-results
[date-format]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/mapping-date-format.html#built-in-date-formats
[date]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/date.html
[date_nanos]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/date_nanos.html

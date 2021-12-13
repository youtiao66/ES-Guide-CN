# Term 精确值查询

返回包含指定字段**精确**值的文档

使用 `term` 查询去查找基于指定精确值的文档，例如价格，产品ID，或者用户名

**警告：**

- 避免使用 `term` 查询 `text` 字段
- 默认情况下，Elasticsearch 会对 `text` 字段进行分词分析。这会导致对 `text` 字段精确匹配异常困难
- 搜索 `text` 字段，请用 `match` 查询代替

## 请求示例

```js
GET /_search
{
  "query": {
    "term": {
      "user.id": {
        "value": "kimchy",
        "boost": 1.0
      }
    }
  }
}
```

## 顶层参数

`<field>`

(必选, `object`) 想要搜索的字段

### `<field>` 参数

- `value` (必须, `string`) <br>
想要精确查找的字段。必须精确匹配该字段值，包括空白符和大小写。

- `boost` (可选, `float`) <br>
用浮点数去指定查询的相关性评分。默认值 `1.0`. <br>
可以使用 `boost` 参数去调整包含两个或者更多查询的评分. <br>
相对于默认值 `1.0`。`0-1.0` 之间降低评分。大于 `1.0` 提高评分

- `case_insensitive` [7.10.0] (可选, `Boolean`) <br>
通过设置该值为 `true` 实现对指定字段值大小写不敏感匹配。默认值为 `false` 代表大小写敏感，同时取决于实际字段的映射关系

## 说明

### 避免使用 `term` 查询 `text` 字段

默认情况下，Elasticsearch 在分析的过程中会改变 `text` 字段的值。例如，默认的标准分析会如下改变 `text` 字段值：

- 移除大部分标点符号
- 把剩下的内容拆分为独立的词语，叫做 `tokens`
- 把这些 `tokens` 转换为小写

为了优化 `text` 字段的搜索，`match` 查询也会在搜索之前分析指定的查询关键字。这意味着 `match` 查询可以通过分析后的 `tokens` 搜索 `text` 字段，而不是通过精确的关键字

`term` 查询**不分析**搜索的字段。`term` 查询仅搜索精确值。这就意味着 `term` 查询在搜索 `text` 字段时可能无法获得期望的结果

为了感受搜索结果的区别，请尝试一下示例：

**1. 通过 `full_text` 创建一个 `text` 字段索引**

```js
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "full_text": { "type": "text" }
    }
  }
}
```

**2. 添加一个 `full_text` 字段值为 `Quick Brown Foxes!` 的文档**

```js
PUT my-index-000001/_doc/1
{
  "full_text":   "Quick Brown Foxes!"
}
```

因为 `full_text` 是 `text` 字段，Elasticsearch 在分析的过程中把 `Quick Brown Foxes!` 转换为 `[quick, brown, fox]`

**3. 用 `term` 查询去搜索 `full_text` 字段的 `Quick Brown Foxes!`。加上 `pretty` 参数去美化返回值。**

```js
GET my-index-000001/_search?pretty
{
  "query": {
    "term": {
      "full_text": "Quick Brown Foxes!"
    }
  }
}
```

因为 `full_text` 字段不再包含精确值 `Quick Brown Foxes!`, `term` 搜索结果为空

**4. 用 `match` 去搜索 `full_text` 字段的 `Quick Brown Foxes!` 值。**

```js
GET my-index-000001/_search?pretty
{
  "query": {
    "match": {
      "full_text": "Quick Brown Foxes!"
    }
  }
}
```

不像 `term`, `match` 在查询之前会对搜索的字段值 `Quick Brown Foxes!` 进行分析。`match` 查询会得到 `full_text` 字段包含 `quick`, `brown` 或者 `fox` tokens 的文档。

以下就是 `match` 查询的搜索结果

```js
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.8630463,
    "hits" : [
      {
        "_index" : "my-index-000001",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.8630463,
        "_source" : {
          "full_text" : "Quick Brown Foxes!"
        }
      }
    ]
  }
}
```


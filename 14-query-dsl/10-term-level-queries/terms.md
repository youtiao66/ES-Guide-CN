# Terms 多个精确值查询

返回指定字段包含一个或者多个**精确**值的文档

除可以搜索多个值以外，多个精确值查询基本和精确值查询相同

## 请求示例
以下搜索返回`user.id`包含`kimchy`或者`elkbee`的文档

```js
GET /_search
{
  "query": {
    "terms": {
      "user.id": [ "kimchy", "elkbee" ],
      "boost": 1.0
    }
  }
}
```

## 顶层参数

| 参数      | 类型   | 说明                                                             |
|-----------|--------|------------------------------------------------------------------|
| `<field>` | object | 必填，想要搜索的字段                                             |
| `boost`   | float  | 用浮点数去指定查询的[相关性评分][relevance-scores]。默认值 `1.0` |

参数的值是想要搜索的词数组。搜索结果必须精确匹配一个或者多个值，包括空白符和大小写。

默认情况下，多个精确值查询最高限制 65,536 个。可以通过 [`index.max_terms_count`][max_terms_count] 配置修改该限制

> 💬**说明：** 想要通过已经存在文档字段值作为搜索词组，请使用[多个精确值查询表][terms-lookup]参数



## 说明
### 多个精确值查询高亮

[高亮][highlighting] 是最优解. 多个精确值查询根据如下因素返回高亮的结果

- 高亮类型
- 查询的精确值数量

### 多个精确值查询表

多个精确值查询表先拉取已存在文档的字段精确值。然后以这些值作为搜索精确值，这有助于搜索大量的精确值。

激活多个精确值查询表必须设置`_source`参数。不能在远程索引上使用跨集群搜索多个精确值查询表

**多个精确值查询表参数**

| 参数    | 类型   | 说明                                                                                          |
|---------|--------|-----------------------------------------------------------------------------------------------|
| index   | string | 必填，索引名称                                                                                |
| id      | string | 必填，文档 [ID][id-field]                                                                     |
| path    | string | 必填，字段名称，用这些值作为搜索精确值。如果字段值包括嵌套的子对象数组，可以通过`.`点语法访问 |
| routing | string | 可选，自定义拉取精确值文档的[路由][routing-field]。如果时自定义索引文档路由，该参数必选       |

**多个精确值查询表示例**

**1. 为`full_text`字段创建一个叫做 `color` 的字段索引**

```js
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "color": { "type": "keyword" }
    }
  }
}
```

**2. 添加一个 `color` 字段值为 `["blue", "green"]` 的文档1**

```js
PUT my-index-000001/_doc/1
{
  "full_text":   "Quick Brown Foxes!"
}
```

**3. 添加一个 `color` 字段值为 `"blue"` 的文档2**

```js
PUT my-index-000001/_doc/2
{
  "color":   "blue"
}
```

**4. 用 `term` 查询去搜索 `full_text` 字段的 `Quick Brown Foxes!`。加上 `pretty` 参数去美化返回值。**

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

**5. 通过多个精确值查询表参数查找包含一个或者多个相同精确值的文档2. 美化后响应值可读性更好**

```js
GET my-index-000001/_search?pretty
{
  "query": {
    "terms": {
        "color" : {
            "index" : "my-index-000001",
            "id" : "2",
            "path" : "color"
        }
    }
  }
}
```

因为文档1和文档2都包含颜色`blue`, 两个文档都会被返回

```js
{
  "took" : 17,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "my-index-000001",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "color" : [
            "blue",
            "green"
          ]
        }
      },
      {
        "_index" : "my-index-000001",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "color" : "blue"
        }
      }
    ]
  }
}
```

[max_terms_count]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/index-modules.html#index-max-terms-count
[terms-lookup]: https://youtiao66.gitbook.io/es-guide-cn/query-dsl/term-level-queries/terms#duo-ge-jing-que-zhi-cha-xun-biao
[relevance-scores]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-filter-context.html#relevance-scores
[highlighting]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/highlighting.html
[id-field]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/mapping-id-field.html
[routing-field]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/mapping-routing-field.html

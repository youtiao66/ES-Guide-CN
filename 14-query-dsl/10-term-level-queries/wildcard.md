# Wildcard 通配符查询

返回包含匹配指定通配符的文档

通配符就是匹配一个或多个字符的占位符。例如`*`通配符匹配零个或者多个字符。通过通配符和其它字符组合构建通配符模式。



## 请求示例

以下搜索返回`user.id`字段包含以`ki`开始并且以`y`结束词语的文档。这些匹配的词语包括`kiy`, `kity`或者`kimchy`.

```js
GET /_search
{
  "query": {
    "wildcard": {
      "user.id": {
        "value": "ki*y",
        "boost": 1.0,
        "rewrite": "constant_score"
      }
    }
  }
}
```



## 顶层参数

`<field>`

(必选, `object`) 想要搜索的字段

### `<field>` 参数

- `boost` (可选, `float`) <br>
用浮点数去指定查询的相关性评分。默认值 `1.0`. <br>
可以使用 `boost` 参数去调整包含两个或者更多查询的评分. <br>
相对于默认值 `1.0`。`0-1.0` 之间降低评分。大于 `1.0` 提高评分

- `case_insensitive` [7.10.0] (可选, `Boolean`) <br>
通过设置该值为 `true` 实现对指定字段值大小写不敏感匹配。默认值为 `false` 代表大小写敏感，同时取决于实际字段的映射关系

- `rewrite` (可选, `string`) <br>
重写查询的方法。有效值和更多信息请参考 [`rewrite` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-multi-term-rewrite.html)

- `value` (必须, `string`) <br>
想要查询的通配符模式。 <br>
该参数支持两种通配符： <br>
  * `?`, 匹配任何单个字符
  * `*`, 匹配零个或者多个字符，可以不匹配字符

> ⚠️警告：通配符模式避免以`*`或者`?`开始。这样有可能增加搜索词语的迭代次数和降低搜索性能。



## 说明

### 允许低效查询

如果`search.allow_expensive_queries`设置为`false`, 通配符查询将不会执行

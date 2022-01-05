# Match 匹配查询

返回匹配指定文本，数字，日期或者布尔值的文档。该文本在匹配之前会进行词法分析。

`match`查询是执行全文搜索的标准查询，包括模糊查询选项。

## 请求示例

```js
GET /_search
{
  "query": {
    "match": {
      "message": {
        "query": "this is a test"
      }
    }
  }
}
```

## 顶层参数

| 参数  | 类型   | 必填 | 说明           |
|-------|--------|------|----------------|
| field | object | 是   | 想要搜索的字段 |

## `field`参数

| 参数                                       | 类型                        | 必填 | 说明                                                                                                    |
|--------------------------------------------|-----------------------------|------|---------------------------------------------------------------------------------------------------------|
| query                                      | text, number, boolean, date | 是   | 想要搜索的值, 1️⃣                                                                                         |
| analyzer                                   | string                      | 否   | 把查询文本转换为 tokens 使用的分析器，默认为字段映射中指定的分析器，若映射中未指定则为索引的默认分析器  |
| auto_generate_synonyms_phrase_queryanalyze | boolean                     | 否   | 默认值为`true`, 将自动创建多个近义词的短语查询，参考[近义词][近义词]示例                                |
| fuzziness                                  | string                      | 否   | 允许匹配的最大编辑距离，具体信息参考[Fuzziness][Fuzziness], 示例参考[匹配查询中的模糊值][fuzziness]     |
| max_expansions                             | integer                     | 否   | 查询扩展词语的最大数值，默认值`50`                                                                      |
| prefix_length                              | integer                     | 否   | 模糊匹配从左侧起始不改变的字符长度，默认值`0`                                                           |
| fuzzy_transpositions                       | boolean                     | 否   | 如果为`true`, 模糊查询时两个相邻字符可以换位 (ab → ba), 默认值`true`                                    |
| fuzzy_rewrite                              | string                      | 否   | 用于重写查询的方法，详情参考[rewrite parameter][rewrite-parameter], 2️⃣                                   |
| lenient                                    | boolean                     | 否   | 如果为`true`, 格式化错误将被忽略，例如使用文本值查询数字字段，默认值`false`                             |
| operator                                   | string                      | 否   | 用于解释文本查询值的是非逻辑                                                                            |
| minimum_should_match                       | string                      | 否   | 返回文档必须匹配的语句最小数量，详情参考[minimum_should_match parameter][minimum_should_match]          |
| zero_terms_query                           | string                      | 否   | 如果分析器移除了所有 tokens, 指明是否不返回任何文档，例如当使用`stop`筛选时，示例参考[零词语查询][zero] |

1. `query` - 匹配查询会在执行搜索之前对文本进行此法分析。因此，匹配查询对文本字段搜索的是分析后的 tokens 而不是精确值
2. `fuzzy_rewrite` - 如果`fuzziness`参数不是`0`, `fuzzy_rewrite`默认重写方法为`top_terms_blended_freqs_${max_expansions}`

**operator**

| 合法值 | 说明                                                                           |
|--------|--------------------------------------------------------------------------------|
| OR     | **默认值**，例如查询值为`capital of Hungary`被解释为`capital OR of OR Hungary` |
| AND    | 例如查询值为`capital of Hungary`被解释为`capital AND of AND Hungary`           |



[近义词]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-match-query.html#query-dsl-match-query-synonyms
[Fuzziness]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/common-options.html#fuzziness
[fuzziness]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-match-query.html#query-dsl-match-query-fuzziness
[rewrite-parameter]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-multi-term-rewrite.html
[minimum_should_match]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-minimum-should-match.html
[zero]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-match-query.html#query-dsl-match-query-zero

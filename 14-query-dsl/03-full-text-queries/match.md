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

| 参数  | 类型   | 说明                 |
|-------|--------|----------------------|
| field | object | 必填，想要搜索的字段 |

## `field`参数

| 参数                                       | 类型                     | 说明                                                                                                    |
|--------------------------------------------|--------------------------|---------------------------------------------------------------------------------------------------------|
| query                                      | text,number,boolean,date | 必填，想要搜索的值, 1️⃣                                                                                   |
| analyzer                                   | string                   | 把查询文本转换为 tokens 使用的分析器，默认为字段映射中指定的分析器，若映射中未指定则为索引的默认分析器  |
| auto_generate_synonyms_phrase_queryanalyze | boolean                  | 默认值为`true`, 将自动创建多个近义词的短语查询，参考[近义词][近义词]示例                                |
| fuzziness                                  | string                   | 允许匹配的最大编辑距离，具体信息参考[Fuzziness][Fuzziness], 示例参考[匹配查询中的模糊性][fuzziness]     |
| max_expansions                             | integer                  | 查询扩展词语的最大数值，默认值`50`                                                                      |
| prefix_length                              | integer                  | 模糊匹配从左侧起始不改变的字符长度，默认值`0`                                                           |
| fuzzy_transpositions                       | boolean                  | 如果为`true`, 模糊查询时两个相邻字符可以换位 (ab → ba), 默认值`true`                                    |
| fuzzy_rewrite                              | string                   | 用于重写查询的方法，详情参考[rewrite parameter][rewrite-parameter], 2️⃣                                   |
| lenient                                    | boolean                  | 如果为`true`, 格式化错误将被忽略，例如使用文本值查询数字字段，默认值`false`                             |
| operator                                   | string                   | 用于解释文本查询值的是非逻辑                                                                            |
| minimum_should_match                       | string                   | 返回文档必须匹配的语句最小数量，详情参考[minimum_should_match parameter][minimum_should_match]          |
| zero_terms_query                           | string                   | 如果分析器移除了所有 tokens, 指明是否不返回任何文档，例如当使用`stop`筛选时，示例参考[零词语查询][zero] |

1. `query` - 匹配查询会在执行搜索之前对文本进行此法分析。因此，匹配查询对文本字段搜索的是分析后的 tokens 而不是精确值
2. `fuzzy_rewrite` - 如果`fuzziness`参数不是`0`, `fuzzy_rewrite`默认重写方法为`top_terms_blended_freqs_${max_expansions}`

**operator**

| 合法值 | 说明                                                                           |
|--------|--------------------------------------------------------------------------------|
| OR     | **默认值**，例如查询值为`capital of Hungary`被解释为`capital OR of OR Hungary` |
| AND    | 例如查询值为`capital of Hungary`被解释为`capital AND of AND Hungary`           |



## 说明
### 简单示例
通过`<field>`和`query`参数简化匹配查询语法，例如：

```js
GET /_search
{
  "query": {
    "match": {
      "message": "this is a test"
    }
  }
}
```

### 匹配查询如何生效
匹配查询是一种布尔查询，这就意味着提供的文本会被分析然后由分析处理的结果组成一个布尔查询, `operator`参数可以设置为`or`或者`and`用于控制布尔语句（默认值`or`）。通过设置[minimum_should_match parameter][minimum_should_match]参数设置返回文档必须匹配的语句最小数量。

`operator`参数示例如下：

```js
GET /_search
{
  "query": {
    "match": {
      "message": {
        "query": "this is a test",
        "operator": "and"
      }
    }
  }
}
```

分析处理文本的分析器可以自定义。默认为该字段映射中显式定义的分析器，若未显式定义则为默认搜索分析器。

`lenient`参数如果为`true`, 格式化错误将被忽略，例如使用文本值查询数字字段，默认值`false`

### 匹配查询中的模糊性
`fuzziness`允许基于被查询字段类型的模糊匹配，参考[Fuzziness][Fuzziness]. 

`prefix_length`和`max_expansions`参数可以控制模糊处理。如果`fuzziness`参数不是`0`, `fuzzy_rewrite`默认重写方法为`top_terms_blended_freqs_${max_expansions}`用于重写查询的方法。

默认允许模糊换位 (ab → ba). 可以通过`fuzzy_transpositions`设置为`false`禁用

> 💬**说明：** 模糊匹配不可以用于近义词精确匹配或者在相同位置分析过程生成多个 tokens 的情况。在底层实现上，这些词语被扩展为一个特殊的经过协调词语重复率的近义词查询，这就不支持模糊扩展。

```js
GET /_search
{
  "query": {
    "match": {
      "message": {
        "query": "this is a testt",
        "fuzziness": "AUTO"
      }
    }
  }
}
```

### 零词语查询
如果在查询中采用的分析器移除了所有 tokens 例如当使用`stop`筛选时，默认表现为不匹配任何文档。如果想要改变这一点可以使用`zero_terms_query`参数，它接受`none`（默认值）和`all`-对应匹配所有查询

```js
GET /_search
{
  "query": {
    "match": {
      "message": {
        "query": "to be or not to be",
        "operator": "and",
        "zero_terms_query": "all"
      }
    }
  }
}
```

### 近义词
匹配查询支持通过 [synonym_graph][synonym_graph] token 过滤器实现多个词语的近义词扩展。当使用该过滤器时，解析器将为每个近义词创建一个短语匹配查询。例如，近义词`ny, new york`将生成：

```
(ny OR ("new york"))
```

这也可能用连词替代匹配多个词语的近义词

```js
GET /_search
{
   "query": {
       "match" : {
           "message": {
               "query" : "ny city",
               "auto_generate_synonyms_phrase_query" : false
           }
       }
   }
}
```

以上示例将创建布尔查询

```
(ny OR (new AND york)) city
```

该示例将匹配词语`ny`或者连词`new AND york`. `auto_generate_synonyms_phrase_query`参数默认值为`true`



[近义词]: https://youtiao66.gitbook.io/es-guide-cn/query-dsl/full-text-queries/match#jin-yi-ci
[Fuzziness]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/common-options.html#fuzziness
[fuzziness]: https://youtiao66.gitbook.io/es-guide-cn/query-dsl/full-text-queries/match#pi-pei-cha-xun-zhong-de-mo-hu-xing
[rewrite-parameter]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-multi-term-rewrite.html
[minimum_should_match]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-minimum-should-match.html
[zero]: https://youtiao66.gitbook.io/es-guide-cn/query-dsl/full-text-queries/match#ling-ci-yu-cha-xun
[synonym_graph]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/analysis-synonym-graph-tokenfilter.html

# Query DSL 查询表达式

Elasticsearch 提供完整的基于 JSON 领用专用语法 (DSL - Domain Specific Language) 进行查询。查询表达式作为一种抽象语法树 (AST - Abstract Syntax Tree) 由两种类型的子句组成：

- **子查询** <br>
子查询搜索特定字段的特定值，例如`match`, `term`或者`range`. 这些查询可以单独使用

- **复合查询** <br>
复合查询用于包裹子查询和复合查询，并以逻辑方式(例如`bool`, `dis_max`)或者改变默认特性的方式(例如`constant_score`)组合多个查询

查询语句的特性依赖于其使用方式，参考 [query context or filter context](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-filter-context.html)

## 允许低效查询

部分类型的查询由于其实现方式一般情况下执行效率较低，这可能影响到集群的稳定性。这查询分类如下：

- 需要通过线性扫描识别匹配项的查询：
  * [`script` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-script-query.html)
- 有高预加载成本的查询：
  * [`fuzzy` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-fuzzy-query.html) (除 wildcard 字段以外)
  * [`regexp` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-regexp-query.html) (除 wildcard 字段以外)
  * [`prefix` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-prefix-query.html) (除 wildcard 字段以外或者未搭配 [index_prefixes](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/index-prefixes.html))
  * [`wildcard` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-wildcard-query.html) (除 wildcard 字段以外)
  * 对 `text` 和 `keyword` 字段进行 [`range` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-range-query.html)
- [`joining` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/joining-queries.html)
- 对 [deprecated geo-shapes](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/geo-shape.html#prefix-trees) 进行查询
- 每份文档都是高成本的查询
  * [`script_score` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-script-score-query.html)
  * [`percolate` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-percolate-query.html)

以上查询可以通过设置 `search.allow_expensive_queries` 为 `false` (默认值 `true`) 避免

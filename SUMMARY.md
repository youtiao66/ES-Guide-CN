# Elasticsearch 使用指南 [7.15] 中文版
* [开始使用](README.md)

## Set up
* [设置](04-setup/index.md)
* 安装
  * [Windows 通过`.zip`安装](04-setup/01-install-elasticsearch/zip-windows.md)

## Search your data
* [搜索数据](13-search-your-data/index.md)
* [搜索结果排序](13-search-your-data/sort-search-results.md)

## Query DSL
* [查询表达式](14-query-dsl/index.md)
* [组合查询](14-query-dsl/compound-queries.md)
  * [Boolean 布尔查询](14-query-dsl/02-compound-queries/bool.md)
* [全文查询](14-query-dsl/full-text-queries.md)
  * [Match 匹配查询](14-query-dsl/03-full-text-queries/match.md)
  * [Match phrase 短语匹配](14-query-dsl/03-full-text-queries/match-phrase.md)
* [全匹配查询](14-query-dsl/match-all-query.md)
* [词级查询](14-query-dsl/term-level-queries.md)
  * [Exists 存在查询](14-query-dsl/10-term-level-queries/exists.md)
  * [Range 范围查询](14-query-dsl/10-term-level-queries/range.md)
  * [Term 精确值查询](14-query-dsl/10-term-level-queries/term.md)
  * [Terms 多个精确值查询](14-query-dsl/10-term-level-queries/terms.md)
  * [Wildcard 通配符查询](14-query-dsl/10-term-level-queries/wildcard.md)

## Aggregations
* [聚合](15-aggregations/index.md)
* [桶聚合](15-aggregations/bucket.md)
  * [多值聚合](15-aggregations/01-bucket/terms-aggregation.md)
* [指标聚合](15-aggregations/metrics.md)
  * [基数聚合](15-aggregations/02-metrics/cardinality-aggregation.md)
  * [总数聚合](15-aggregations/02-metrics/sum-aggregation.md)
  * [值计数聚合](15-aggregations/02-metrics/valuecount-aggregation.md)

## Scripting
* [脚本](18-scripting/index.md)
* [Painless 脚本语言](18-scripting/painless.md)
* [如何编写脚本](18-scripting/using.md)

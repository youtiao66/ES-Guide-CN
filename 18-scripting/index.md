# 脚本
在 Elasticsearch 中可以通过脚本计算自定义表达式。例如，可以使用脚本返回计算值作为字段或者评估查询的自定义评分。

默认脚本语言是 [painless][painless]. 可以通过添加`lang`插件使用其他语言编写并执行脚本。可以在任何执行脚本的地方指定语言。

## 可用的脚本语言
Painless 是为 Elasticsearch 设计的，是灵活性的，也可以在脚本 APIs 中用作其他目的。其他语言灵活性欠缺一些，但也可以用于指定目的。

| 语言                     | 沙盒 | 依赖插件   | 目的                    |
|--------------------------|------|------------|-------------------------|
| [painless][painless]     | ✅    | 自带       | 专为 Elasticsearch 设计 |
| [expression][expression] | ✅    | 自带       | 自定义排名和排序更快    |
| [mustache][mustache]     | ✅    | 自带       | 模板                    |
| [java][java]             | ❌    | 自定义添加 | 专业级 API              |



[painless]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/modules-scripting-painless.html
[expression]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/modules-scripting-expression.html
[mustache]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/search-template.html
[java]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/modules-scripting-engine.html

# 如何编写脚本
无论在 Elasticsearch APIs 支持脚本的任何地方，语法模式都是相同的。指定脚本语言，实现脚本逻辑或源代码，添加传递的参数到脚本中：

```json
"script": {
  "lang":   "...",
  "source" | "id": "...",
  "params": { ... }
}
```

| 参数       | 说明                                                                                                                       |
|------------|----------------------------------------------------------------------------------------------------------------------------|
| lang       | 指定脚本语言，默认`painless`                                                                                               |
| source, id | 脚本内容, `source`指定行内脚本, `id`指定已存储的脚本，使用 [stored script APIs][stored-script-apis] 创建和管理已存储的脚本 |
| params     | 指定传递到脚本中作为变量的命名参数, [Use parameters][prefer-params] 代替硬编码变量有助于降低编译时间                       |



## 编写第一个脚本
[Painless][painless] 是默认脚本语言。具有安全，高性能和简洁的特性. 

Painless 脚本由一个或者多个语句构成，可以在开始添加一个或多个用户自定义函数。一个脚本至少包含一个语句。

[Painless execute API][painless-execute-api] 提供通过用户自定义参数和接受值的能力测试脚本。让我们开始一个完整的脚本并回顾他的组成要素。

首先，建立一个字段索引方便有数据进行操作：

```json
PUT my-index-000001/_doc/1
{
  "my_field": 5
}
```

构建一个操作该字段并在查询中执行的脚本。以下查询使用搜索 API 的 [script_fields][script-fields] 参数去检查脚本值。这发生了很多，为了单独地理解我们把它拆分为组件。现在仅需要理解脚本使用`my_field`并操作它。

```json
GET my-index-000001/_search
{
  "script_fields": {
    "my_doubled_field": {
      "script": { 
        "source": "doc['my_field'].value * params['multiplier']", 
        "params": {
          "multiplier": 2
        }
      }
    }
  }
}
```

在大多数情况下脚本是一个标注的 JSON 对象定义的。该对象需要`source`定义脚本内容。没有指定脚本语言所以默认是 Painless



## 在脚本中使用参数
这是在 Elasticsearch 中第一次出现脚本，这个脚本被编译然后存储在缓存中。编译是非常耗费性能的过程。使用命名参数传递比硬编码值更好。

例如，在上一个脚本中可以使用硬编码写出一个看起来更简单的脚本。检索`my_field`的值然后乘以2.

```json
"source": "return doc['my_field'].value * 2"
```

尽管它也能生效，但这个解决方案是相当死板的。我们必须修改脚本源代码去改变乘数，每次乘数改变的时候都要重新编码。

用命名参数替代硬编码会更加灵活，当脚本运行时也可以减少运行时间。现在可以改变乘数而不需要重新编译脚本。

```json
"source": "doc['my_field'].value * params['multiplier']",
"params": {
  "multiplier": 2
}
```

在大多数背景下，默认每 5 分钟可以编译 75 个脚本。默认脚本编译效率是无限制的。该设置可以通过`script.context.$CONTEXT.max_compilations_rate`动态修改。例如，以下设置限制脚本编译每 10 分钟编译 100 个脚本 [field context][field-context]。

```
script.context.field.max_compilations_rate=100/10m
```

> ⚠️**重要：** 如果在短时间内编译太多脚本，将抛出新的动态脚本错误`circuit_breaking_exception`



## 简化脚本
Painless 原生支持语法能力，你可以减少脚本中的冗长和让脚本更简洁。以下就是一个简单示例脚本：

```json
GET my-index-000001/_search
{
  "script_fields": {
    "my_doubled_field": {
      "script": {
        "lang":   "painless",
        "source": "return doc['my_field'].value * params.get('multiplier');",
        "params": {
          "multiplier": 2
        }
      }
    }
  }
}
```

让我们来看看优化后的脚本相对于前一个版本有哪些提高

```json
GET my-index-000001/_search
{
  "script_fields": {
    "my_doubled_field": {
      "script": {
        "source": "doc['my_field'].value * params['multiplier']",
        "params": {
          "multiplier": 2
        }
      }
    }
  }
}
```

这个版本显著地移除了多个组件并简化了语法：

- `lang`语言声明。因为 Painless 是默认脚本语言，如果使用该语言无需指定。
- `return`关键字。Painless 在脚本中自动根据上下文使用最后一个语句生成返回值。
- `get`方法可以通过中括号`[]`替代。Painless 在特定情况下使用快捷方式访问映射类型，可以通过中括号`[]`替代更长的`get`方法。
- `source`语句末尾的分号。在块的最后一个语句不需要分号。然而，在其他情况下确实要求分号消除歧义。

尽可能地使用支持的缩写语法。例如当你创建 [runtime fields][runtime-mapping-fields]



## 存储和检索脚本
可以通过使用 [stored script APIs][stored-script-apis] 存储和检索集群中的脚本。存储脚本减少编译时间和让搜索更快。

> 💬**说明：** 和普通脚本不同，存储脚本需要使用`lang`参数指定脚本语言。

使用 [create stored script API][create-stored-script-api] 创建脚本。例如，以下请求创建一个叫做`calculate-score`的存储脚本。

```json
POST _scripts/calculate-score
{
  "script": {
    "lang": "painless",
    "source": "Math.log(_score * 2) + params['my_modifier']"
  }
}
```

通过 [get stored script API][get-stored-script-api] 检索脚本。

```
GET _scripts/calculate-score
```

在查询中使用脚本，只需要声明时的脚本`id`

```json
GET my-index-000001/_search
{
  "query": {
    "script_score": {
      "query": {
        "match": {
            "message": "some message"
        }
      },
      "script": {
        "id": "calculate-score", 
        "params": {
          "my_modifier": 2
        }
      }
    }
  }
}
```

通过提交 [delete stored script API][delete-stored-script-api] 请求删除已存储脚本

```
DELETE _scripts/calculate-score
```



## 通过脚本更新文档
可以通过指定脚本使用 [update API][docs-update] 更新文档。脚本可以更新、删除或者跳过修改文档。更新 API 也支持传递合并到已存在文档的一部分文档。

首先，让我们索引一个简单的文档：

```json
PUT my-index-000001/_doc/1
{
  "counter" : 1,
  "tags" : ["red"]
}
```

为了增加 counter, 你可以用以下脚本提交一个更新请求：

```json
POST my-index-000001/_update/1
{
  "script" : {
    "source": "ctx._source.counter += params.count",
    "lang": "painless",
    "params" : {
      "count" : 4
    }
  }
}
```

相似地，可以使用更新脚本向标签列表中添加一个标签。因为这只是一个列表，所以即使标签存在，也会添加标签：

```json
POST my-index-000001/_update/1
{
  "script": {
    "source": "ctx._source.tags.add(params['tag'])",
    "lang": "painless",
    "params": {
      "tag": "blue"
    }
  }
}
```

也可以移除标签列表中的标签, Java `List` 的 `remove` 方法在 Painless 中也是可用的。需要想要移除元素的索引。为了避免可能的运行时错误，首先需要保证该标签存在。如果列表包含重复的标签，脚本至少需要移除一个。

```json
POST my-index-000001/_update/1
{
  "script": {
    "source": "if (ctx._source.tags.contains(params['tag'])) { ctx._source.tags.remove(ctx._source.tags.indexOf(params['tag'])) }",
    "lang": "painless",
    "params": {
      "tag": "blue"
    }
  }
}
```

也可以添加和移除文档中的字段。例如，以下脚本添加了字段`new_field`:

```json
POST my-index-000001/_update/1
{
  "script" : "ctx._source.new_field = 'value_of_new_field'"
}
```

相反，以下脚本移除字段`new_field`:

```json
POST my-index-000001/_update/1
{
  "script" : "ctx._source.remove('new_field')"
}
```

还可以更改脚本中执行的操作，而不是更新文档。例如，如果`tags`字段包含`green`以下请求删除文档。否则，什么也不做。

```json
POST my-index-000001/_update/1
{
  "script": {
    "source": "if (ctx._source.tags.contains(params['tag'])) { ctx.op = 'delete' } else { ctx.op = 'none' }",
    "lang": "painless",
    "params": {
      "tag": "green"
    }
  }
}
```



[painless]: https://youtiao66.gitbook.io/es-guide-cn/scripting/painless

[stored-script-apis]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/script-apis.html#stored-script-apis
[prefer-params]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/modules-scripting-using.html#prefer-params
[painless-execute-api]: https://www.elastic.co/guide/en/elasticsearch/painless/7.15/painless-execute-api.html
[script-fields]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/search-fields.html#script-fields
[field-context]: https://www.elastic.co/guide/en/elasticsearch/painless/7.15/painless-field-context.html
[runtime-mapping-fields]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/runtime-mapping-fields.html
[create-stored-script-api]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/create-stored-script-api.html
[get-stored-script-api]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/get-stored-script-api.html
[delete-stored-script-api]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/delete-stored-script-api.html
[docs-update]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/docs-update.html

# 更新映射 APIs
向已经存在的数据流或者索引添加新字段。也可以使用该 API 改变已经存在字段的搜索设置

对于数据流，默认这些改动也可以应用于所有支持索引

```json
PUT /my-index-000001/_mapping
{
  "properties": {
    "email": {
      "type": "keyword"
    }
  }
}
```

## 请求

`PUT /<target>/_mapping`

## 路径参数

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `<target>` | string | 必填，数据流、索引或者别名。也可以是用逗号分隔的列表，支持通配符。通过省略该参数或者使用`*`, `_all`的方式对所有生效 |

## 请求体

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `properties` | [mapping object][mapping] | 必填，字段映射。 |

对于新字段，映射必须包含以下内容：

- 字段名称
- [字段数据类型][mapping-types]
- [映射参数][mapping-params]

对于已经存在的字段，请参考[updating-field-mappings][updating-field-mappings]

## 示例
### 单个目标
更新映射 API 要求一个存在的数据流或者索引。首先，创建一个没有映射的`publications`索引：

```
PUT /publications
```

然后，使用更新映射 API 添加一个新的文本字段`title`, 到`publications`索引中去

```
PUT /publications/_mapping
{
  "properties": {
    "title":  { "type": "text"}
  }
}
```

### 多个目标
更新映射 API 可以通过一次请求应用于多个数据流或者索引。例如，可以同时更新`my-index-000001`和`my-index-000002`索引：

```json
// 创建两个索引
PUT /my-index-000001
PUT /my-index-000002

// 一起更新映射
PUT /my-index-000001,my-index-000002/_mapping
{
  "properties": {
    "user": {
      "properties": {
        "name": {
          "type": "keyword"
        }
      }
    }
  }
}
```

### 向已存在对象字段添加新属性
更新映射 API 可以向已存在对象字段添加新属性。具体参考以下示例：

首先创建一个包含`name`对象字段并且有`first`文本字段子属性的索引

```json
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "name": {
        "properties": {
          "first": {
            "type": "text"
          }
        }
      }
    }
  }
}
```

然后，使用更新映射 API 添加一个新的子属性`last`文本字段到`name`字段

```json
PUT /my-index-000001/_mapping
{
  "properties": {
    "name": {
      "properties": {
        "last": {
          "type": "text"
        }
      }
    }
  }
}
```

### 向已存在对象字段添加多个字段
[Multi-fields][multi-fields] 使得以不同的方式索引相同字段成为可能。使用更新映射 API 更新字段映射参数和对已存在的字段激活 multi-fields

以下示例将展示它是如何使用的：

首先，创建一个带`city`文本字段的索引

```json
PUT /my-index-000001/_mapping
{
  "properties": {
    "name": {
      "properties": {
        "last": {
          "type": "text"
        }
      }
    }
  }
}
```

当文本字段对全文搜索生效时，关键字字段不被分析并且排序和聚合的效果可能会更好。

使用更新映射 API 激活`city`字段的 multi-fields. 以下请求添加`city.raw`关键字 multi-fields, 这将有助于排序

```json
PUT /my-index-000001/_mapping
{
  "properties": {
    "city": {
      "type": "text",
      "fields": {
        "raw": {
          "type": "keyword"
        }
      }
    }
  }
}
```

### 改变已存在字段支持的映射参数
每个映射参数的说明文档展示了是否可以通过更新映射 API 对已存在字段更新它。例如，使用更新映射 API 更新`ignore_above`参数

以下示例将展示它是如何使用的：

首先，创建一个包含`user_id`关键字字段的索引。该字段拥有等于`20`的`ignore_above`参数

```json
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "user_id": {
        "type": "keyword",
        "ignore_above": 20
      }
    }
  }
}
```

然后，使用更新映射接口改变该参数的值到`100`

```json
PUT /my-index-000001/_mapping
{
  "properties": {
    "user_id": {
      "type": "keyword",
      "ignore_above": 100
    }
  }
}
```

### 改变已存在字段映射
除了已支持的映射参数以外，不能改变已存在字段的映射或者字段类型。改变已存在字段可能导致已存储的数据失效。

如果需要改变数据流支持索引中字段的映射，请参考[Change mappings and settings for a data stream][data-streams-change-mappings-and-settings]

如果需要改变其它索引中字段的映射，首先创建一个正确映射的索引，然后把数据重新存储([reindex][docs-reindex])到这个索引中。

以下示例将展示它是如何使用的：

创建一个包含`long`字段类型的`user_id`字段的索引

```json
PUT /my-index-000001
{
  "mappings" : {
    "properties": {
      "user_id": {
        "type": "long"
      }
    }
  }
}
```

通过`user_id`字段值索引多个文档。

```json
POST /my-index-000001/_doc?refresh=wait_for
{
  "user_id" : 12345
}

POST /my-index-000001/_doc?refresh=wait_for
{
  "user_id" : 12346
}
```

为了把`user_id`字段改成关键字字段类型，以正确的映射创建新的索引

```json
PUT /my-new-index-000001
{
  "mappings" : {
    "properties": {
      "user_id": {
        "type": "keyword"
      }
    }
  }
}
```

使用重新索引接口把文档从旧的索引拷贝到新的索引中去

```json
POST /_reindex
{
  "source": {
    "index": "my-index-000001"
  },
  "dest": {
    "index": "my-new-index-000001"
  }
}
```

### 重命名字段
重命名字段会使旧字段名称下已经索引的数据无效。作为替代方案，通过添加别名字段去创建备用字段名

例如，首先创建一个包含`user_identifier`字段的索引

```json
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "user_identifier": {
        "type": "keyword"
      }
    }
  }
}
```

使用更新映射接口为已经存在的`user_identifier`字段添加`user_id`字段别名

```json
PUT /my-index-000001/_mapping
{
  "properties": {
    "user_id": {
      "type": "alias",
      "path": "user_identifier"
    }
  }
}
```



[mapping]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/mapping.html
[mapping-types]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/mapping-types.html
[mapping-params]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/mapping-params.html
[updating-field-mappings]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/indices-put-mapping.html#updating-field-mappings
[multi-fields]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/multi-fields.html
[data-streams-change-mappings-and-settings]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/data-streams-change-mappings-and-settings.html
[docs-reindex]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/docs-reindex.html
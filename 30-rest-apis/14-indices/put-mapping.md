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



[mapping]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/mapping.html
[mapping-types]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/mapping-types.html
[mapping-params]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/mapping-params.html
[updating-field-mappings]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/indices-put-mapping.html#updating-field-mappings

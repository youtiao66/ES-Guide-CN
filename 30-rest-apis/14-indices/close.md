# 关闭索引 API
关闭一个索引

```
POST /my-index-000001/_close
```

## 请求

`POST /<index>/_close`

## 路径参数

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `<index>` | string | 可选，索引名称。也可以是用逗号分隔的列表或者通配符 |

使用`_all`或者`*`, 可以关闭所有索引。通过集群设置[`action.destructive_requires_name`][action-destructive-requires-name]为`true`禁用该功能。可以通过`elasticsearch.yml`文件或者使用[集群更新设置][cluster-update-settings] API 来更新该设置

## 示例
以下示例展示如何关闭一个索引：

```
POST /my-index-000001/_close
```

该 API 返回以下响应：

```json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "indices": {
    "my-index-000001": {
      "closed": true
    }
  }
}
```



[action-destructive-requires-name]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/index-management-settings.html#action-destructive-requires-name
[cluster-update-settings]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/cluster-update-settings.html

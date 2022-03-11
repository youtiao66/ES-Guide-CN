# 打开索引 API
打开一个关闭的索引。对于数据流，该 API 可以打开任何关闭的支持索引。

```
POST /my-index-000001/_open
```

## 请求

`POST /<target>/_open`

## 路径参数

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `<target>` | string | 可选，数据流、索引或者别名。也可以是用逗号分隔的列表，支持通配符 |

使用`_all`或者`*`, 可以关闭所有索引。通过集群设置[`action.destructive_requires_name`][action-destructive-requires-name]为`true`禁用该功能。可以通过`elasticsearch.yml`文件或者使用[集群更新设置][cluster-update-settings] API 来更新该设置

## 示例
以下请求重新打开一个叫做`my-index-000001`的已关闭索引

```
POST /my-index-000001/_open
```

该 API 返回以下响应：

```json
{
  "acknowledged" : true,
  "shards_acknowledged" : true
}
```



[action-destructive-requires-name]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/index-management-settings.html#action-destructive-requires-name
[cluster-update-settings]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/cluster-update-settings.html

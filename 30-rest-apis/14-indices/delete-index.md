# 删除索引 API
删除一个或者多个索引

```
DELETE /my-index-000001
```

## 请求

`DELETE /<index>`

## 路径参数

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `<target>` | string | 必填，删除的索引，不能通过[索引别名][aliases]删除。也可以是用逗号分隔的数组。 |

使用`_all`或者`*`, 可以删除所有索引。通过集群设置[`action.destructive_requires_name`][action-destructive-requires-name]为`true`禁用该功能。



[aliases]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/aliases.html
[action-destructive-requires-name]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/index-management-settings.html#action-destructive-requires-name

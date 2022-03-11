# 创建索引 API
创建一个新的索引

```
PUT /my-index-000001
```

## 请求

`PUT /<index>`

## 路径参数

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `<index>` | string | 必填，你希望创建的所有名称 |

## 响应

```json
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "my-index-000001"
}
```

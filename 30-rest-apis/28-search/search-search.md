# 搜索接口
返回与请求中定义的查询相匹配的搜索命中结果

```json
GET /my-index-000001/_search
```

## 请求
`GET /<target>/_search`

`GET /_search`

`POST /<target>/_search`

`POST /_search`

## 描述
允许执行一个搜索查询然后获取到匹配查询的搜索结果。可以通过查询参数或请求体的方式描述搜索查询。

## 路径参数

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `<target>` | string | 可选，数据流、索引或者别名。支持通配符(`*`), 支持`_all` |

## 示例

```json
GET /my-index-000001/_search?from=40&size=20
{
  "query": {
    "term": {
      "user.id": "kimchy"
    }
  }
}
```

响应结果如下：

```json
{
  "took": 5,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 20,
      "relation": "eq"
    },
    "max_score": 1.3862942,
    "hits": [
      {
        "_index": "my-index-000001",
        "_type" : "_doc",
        "_id": "0",
        "_score": 1.3862942,
        "_source": {
          "@timestamp": "2099-11-15T14:12:12",
          "http": {
            "request": {
              "method": "get"
            },
            "response": {
              "status_code": 200,
              "bytes": 1070000
            },
            "version": "1.1"
          },
          "source": {
            "ip": "127.0.0.1"
          },
          "message": "GET /search HTTP/1.1 200 1070000",
          "user": {
            "id": "kimchy"
          }
        }
      },
      ...
    ]
  }
}
```

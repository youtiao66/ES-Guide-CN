# 查询索引 API
返回一个或者多个索引信息。对于数据流, API 返回有关流的支持索引的信息。

```
GET /my-index-000001
```

## 请求

`GET /<target>`

## 路径参数

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `<target>` | string | 必填，请求的数据流、索引和别名。也可以是用逗号分隔的数组。支持通配符(`*`)。通过省略该参数或者使用`*`, `_all`的方式查询所有 |

## 响应

```json
{
    "my-index-000001": {
        "aliases": {},
        "mappings": {},
        "settings": {
            "index": {
                "routing": {
                    "allocation": {
                        "include": {
                            "_tier_preference": "data_content"
                        }
                    }
                },
                "number_of_shards": "1",
                "provided_name": "my-index-000001",
                "creation_date": "1646732853587",
                "number_of_replicas": "1",
                "uuid": "0ELRXqlTSMK_fkv_8UI_0g",
                "version": {
                    "created": "7150299"
                }
            }
        }
    }
}
```

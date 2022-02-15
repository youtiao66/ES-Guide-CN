# Match all 全匹配查询

## Match all 全匹配查询
最简单的查询，匹配所有文档，给他们`1.0`的评分

```js
GET /_search
{
    "query": {
        "match_all": {}
    }
}
```

可以通过`boost`参数改变评分

```js
GET /_search
{
  "query": {
    "match_all": { "boost" : 1.2 }
  }
}
```

## Match None 零匹配查询
和全匹配查询相反，不匹配任何文档

```js
GET /_search
{
  "query": {
    "match_none": {}
  }
}
```

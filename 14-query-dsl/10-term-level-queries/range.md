# Range 范围查询

返回包含指定范围的文档



## 请求示例

以下搜索返回`age`字段在`10`到`20`之间的文档

```js
GET /_search
{
  "query": {
    "range": {
      "age": {
        "gte": 10,
        "lte": 20,
        "boost": 2.0
      }
    }
  }
}
```



## 顶层参数

`<field>`

(必选, `object`) 想要搜索的字段

### `<field>` 参数

- `gt` (可选) <br>
大于

- `gte` (可选) <br>
大于或等于

- `lt` (可选) <br>
小于

- `lte` (可选) <br>
小于或等于

- `format` (可选, `string`) <br>
查询中需要被转换为`date`值的日期格式 <br>
默认情况下，使用字段映射中提供的 [date `format`](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/mapping-date-format.html). 该值可以覆盖映射的格式 <br>
可用语法请参考 [`format`](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/mapping-date-format.html) <br>
***⚠️警告：*** 如果格式或者日期值不完整，范围查询将使用默认值替换缺失的部分。具体参考 [Missing date components](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-range-query.html#missing-date-components)

- `relation` (可选, `string`) <br>
表明范围查询是如何匹配`range`字段的值。有效值如下： <br>
  * `INTERSECTS` (默认值) <br>
  匹配查询范围的交集
  * `CONTAINS` <br>
  匹配完全包含查询范围
  * `WITHIN` <br>
  匹配完全包含于查询范围

- `time_zone` (可选, `string`) <br>
用于转换日期值 [`UTC`偏移量](https://en.wikipedia.org/wiki/List_of_UTC_time_offsets) 或者 [`IANA`时区](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) <br>
有效值包括 ISO 8601 UTC 偏移量，例如`+01:00`或者`-08:00`. 以及`IANA`时区，例如`America/Los_Angeles` <br>
使用时区参数查询的示例请参考 todo: <br>
***ℹ️提示：*** 时区参数不会影响**不会**影响`now`函数的日期值，`now`函数总是以当前系统时间中 UTC 为准。 <br>
然而，时区参数是通过`now`和 [date math rounding](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/common-options.html#date-math) 计算结果转换日期。例如时区参数将会转换`now/d`的值

- `boost` (可选, `float`) <br>
用浮点数去指定查询的相关性评分。默认值 `1.0`. <br>
可以使用 `boost` 参数去调整包含两个或者更多查询的评分. <br>
相对于默认值 `1.0`。`0-1.0` 之间降低评分。大于 `1.0` 提高评分



## 说明
### 对`text`字段和`keyword`字段使用范围查询
如果`search.allow_expensive_queries`设置为`false`, 对`text`字段和`keyword`字段使用的范围查询将不被执行

### 对`date`字段使用范围查询
当`<field>`参数是一个日期字段，可以使用一下参数：

- gt
- gte
- lt
- lte

例如，以下搜索返回`timestamp`字段包含昨天和今天的。

```js
GET /_search
{
  "query": {
    "range": {
      "timestamp": {
        "gte": "now-1d/d",
        "lt": "now/d"
      }
    }
  }
}
```

### 缺省日期部分
在范围查询和日期范围聚合中，将会用以下的日期替换掉缺省的部分。缺省的年将不被替换。

```yml
MONTH_OF_YEAR:    01
DAY_OF_MONTH:     01
HOUR_OF_DAY:      23
MINUTE_OF_HOUR:   59
SECOND_OF_MINUTE: 59
NANO_OF_SECOND:   999_999_999
```

例如，如果格式是`yyyy-MM`, 大于`2099-12`的值将被转换为`2099-12-01T23:59:59.999_999_999Z`. 日期采用提供的年(2099)和月(12)，与此同时采用默认的日(01), 时(23), 分(59), 秒(59)和纳秒(999_999_999)

### 日期计算和取整
[日期计算](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/common-options.html#date-math) 结果取整参数如下：

| 参数 | 说明 | 示例 |
| --- | --- | --- |
| `gt`  | 向上取整到（原始日期未覆盖的）第一毫秒 | `2014-11-18||/M`向上取整到`2014-12-01T00:00:00.000`, 不包括整个十一月 |
| `gte` | 向下取整到第一毫秒                   | `2014-11-18||/M`向下取整到`2014-11-01T00:00:00.000`, 包括整个月 |
| `lt`  | 向下取整到（原始值之前的）最后一毫秒   | `2014-11-18||/M`向下取整到`2014-10-31T23:59:59.999`, 不包括整个十一月 |
| `lte` | 向上取整到（取整时段内）最后一毫秒     | `2014-11-18||/M`向上取整到`2014-11-30T23:59:59.999`, 包括整个月 |

### 使用时区参数示例
当使用时区参数通过 UTC 偏移量把日期值转换为 UTC. 例如：

```js
GET /_search
{
  "query": {
    "range": {
      "timestamp": {
        "time_zone": "+01:00",        //1️⃣
        "gte": "2020-01-01T00:00:00", //2️⃣
        "lte": "now"                  //3️⃣
      }
    }
  }
}
```

1️⃣ 表明日期值使用`+01:00`的 UTC 偏移量
2️⃣ 通过`+01:00`的 UTC 偏移量，日期被转换为`2019-12-31T23:00:00 UTC`
3️⃣ 时区参数不影响`now`计算结果

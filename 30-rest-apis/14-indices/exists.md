# 是否存在 API
校验数据流、索引或者别名是否存在

```
HEAD my-data-stream
```

## 请求

`HEAD <target>`

## 路径参数

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `<target>` | string | 必填，数据流、索引和别名。也可以是用逗号分隔的列表。支持通配符(`*`) |

## 响应码

| 参数 | 说明 |
| --- | --- |
| `200`| 所有目标存在 |
| `404`| 一个或者多个指定的目标不存在 |

# 启动 Elasticsearch
启动 Elasticsearch 的方式根据安装方式的不同而不同。

## 归档包（`.zip`）
如果在 Windows 上通过`.zip`包的方式安装 Elasticsearch, 可以通过命令行启动 Elasticsearch. 如果希望在没有任何用户交互的情况下开机自启动, [install Elasticsearch as a service][windows-service]

### 通过命令行启动
Elasticsearch 可以通过如下命令行启动：

```
.\bin\elasticsearch.bat
```

如果采用 Elasticsearch 密钥文件的方式，可能会提示输入密钥文件密码。访问 [Secure settings][secure-settings] 了解更多细节

默认情况下，日志打印到控制台(STDOUT)和`logs`目录中的`<clustername>.log`文件。启动时打印一些信息，一旦初始化以后系统将运行在前台并不会继续打印日志直到发生异常。运行时可以通过 HTTP 接口方式(默认端口号 9200)和 Elasticsearch 进行交互。键入`Ctrl-C`终止



[windows-service]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/zip-windows.html#windows-service
[secure-settings]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/secure-settings.html
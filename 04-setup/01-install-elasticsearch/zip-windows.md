# Windows 通过`.zip`安装 Elasticsearch

Windows 可以通过`.zip`包安装 Elasticsearch. 附带有`elasticsearch-service.bat`命令以服务的形式启动

> ℹ️提示： Elasticsearch 一直以来在 Windows 都是通过`.zip`包安装的. Windows 中`MSI`安装包的方式也可以并且安装启动更加简洁. 如果你喜欢，你可以继续使用`.zip`包的方式

这个包既包含免费版本也包含付费订阅功能。所有的付费订阅功能都有 30 天的试用期

最新稳定版 Elasticsearch 可以在[下载页](https://www.elastic.co/cn/downloads/elasticsearch)获取。其他版本可以在[历史发布版本](https://www.elastic.co/cn/downloads/past-releases)也获取。

Elasticsearch 包含附带的来自于 JDK 官方维护者(GPLv2+CE)的[OpenJDK](https://openjdk.java.net/). 使用独立版本的 Java, 请参考 [JVM 版本要求](todo:url)

## 下载和安装`.zip`包

Elasticsearch v7.15.2 `.zip`包下载地址：

[https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.15.2-windows-x86_64.zip](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.15.2-windows-x86_64.zip)

解压缩后会创建`elasticsearch-7.15.2`文件夹，以该目录创建`%ES_HOME%`环境变量。在命令行窗口中，切换到`%ES_HOME%`目录：

```
cd c:\elasticsearch-7.15.2
```

## 开启自动创建系统目录及文件

部分付费功能会自动创建目录及文件。默认情况下，需要配置允许自动创建索引和必须的步骤。然而，如果已经禁用自动创建索引则必须在`elasticsearch.yml`文件中配置`action.auto_create_index`允许付费功能创建以下目录及文件

```yml
action.auto_create_index: .monitoring*,.watches,.triggered_watches,.watcher-history*,.ml*
```

> ⚠️重要： 如果使用[Logstash](https://www.elastic.co/products/logstash)或者[Beats](https://www.elastic.co/products/beats)就需要在`action.auto_create_index`中配置额外的索引名称，具体的值依赖本地配置。如果不确定本地环境的正确配置，可以考虑设置为`*`允许自动创建所有目录及文件

## 通过命令行启动

Elasticsearch 可以通过如下命令行启动：

```
.\bin\elasticsearch.bat
```

如果采用 Elasticsearch 密钥文件的方式，可能会提示输入密钥文件密码。访问[Secure settings](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/secure-settings.html)了解更多细节

默认情况下，日志打印到控制台(STDOUT)和`logs`目录中的`<clustername>.log`文件。启动时打印一些信息，一旦初始化以后系统将运行在前台并不会继续打印日志直到发生异常。运行时可以通过 HTTP 接口方式(默认端口号 9200)和 Elasticsearch 进行交互。键入`Ctrl-C`终止

## 通过命令行配置

Elasticsearch 默认通过`%ES_HOME%\config\elasticsearch.yml`文件加载配置。具体配置参考[Configuring Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/settings.html)

所有配置文件可以指定的配置都可以通过命令行`-E`语法指定：

```
.\bin\elasticsearch.bat -Ecluster.name=my_cluster -Enode.name=node_1
```

> 🏳️说明： 包含空格的值需要添加引号。例如`-Epath.logs="C:\My Logs\logs"`

> ℹ️提示： 跟集群有关的配置(比如`cluster.name`)应该添加到`elasticsearch.yml`配置文件中，而与 Node 关联的配置(比如`node.name`)可以在命令行中指定

## 测试运行是否成功

通过向`localhost:9200`发起 HTTP 请求测试运行是否成功：

```js
GET /
```

运行成功时响应结果如下：

```json
{
  "name" : "Cp8oag6",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "AT69_T_DTp-1qgIJlatQqA",
  "version" : {
    "number" : "7.15.2",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "f27399d",
    "build_date" : "2016-03-30T09:51:41.449Z",
    "build_snapshot" : false,
    "lucene_version" : "8.9.0",
    "minimum_wire_compatibility_version" : "1.2.3",
    "minimum_index_compatibility_version" : "1.2.3"
  },
  "tagline" : "You Know, for Search"
}
```

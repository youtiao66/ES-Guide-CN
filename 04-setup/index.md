# 设置

该部分包含 Elasticsearch 设置和运行信息，主要包括以下部分：

- 下载
- 安装
- 启动
- 配置

## 支持的平台

官方支持的操作系统和`JVMs`支持情况参考：[支持表](https://www.elastic.co/cn/support/matrix). 在列出的系统上 Elasticsearch 已测试通过，在其他系统上也可能正常工作。

## Java(JVM) 版本

Elasticsearch 通过 Java 和 JDK 官方维护者(GPLv2+CE)构建和分发的不同版本[OpenJDK](https://openjdk.java.net/). 推荐使用位于 Elasticsearch 主目录的`jdk`文件夹附带的 JVM.

使用独立版本 Java 需要设置`ES_JAVA_HOME`环境变量。如果使用与附带 JVM 不同版本的 Java, 推荐使用支持的 LTS 版本。如果使用已知的不合格版本 Java, Elasticsearch 将启动失败。使用独立版本 JVM 后附带的 JVM 目录可能被删除。

## 使用专用 hosts

在生产环境中，推荐使用专用的 host 运行 Elasticsearch 或主服务。Elasticsearch的几个特性（如自动调整JVM堆大小）都需要假设它是主机或容器上唯一的资源密集型应用程序。例如，您可以将 Metricbeat 与 Elasticsearch 一起运行，以获取集群统计信息，但资源密集型 Logstash 部署应该在自己的 host 上进行。

# Painless 脚本语言
*Painless* 是一种高性能、安全的专为 Elasticsearch 设计的脚本语言。可以使用 Painless 安全地编写内联脚本，支持在任何地方存储脚本。

Painless 提供众多围绕以下核心准则的能力：

- **安全**：以最高的重要性确保集群的安全。为实现这一目标, Painless 采用细颗粒度的白名单，其颗粒度精确到类的成员。任何不在白名单中的都将编译失败。每个脚本上下文的可用类、方法
和字段的完整列表参考 [Painless API Reference][painless-api-reference]。
- **高性能**：Painless 直接编译为 JVM 字节码能够很好的利用 JVM 提供的优点。此外 Painless 通常能够避免在运行时依赖额外较慢的检查功能。
- **简洁**：Painless 实现的语法对任何有基本编程经验的人非常友好。采用 Java 语法的子集同时添加了一些额外的特性以增强可读性和移除样板。

## 开始编辑脚本
准备好使用 Painless 编写脚本了吗？请学习 [write your first script][modules-scripting-using]

如果你早已熟悉 Painless. 请查看 [Painless Language Specification][painless-lang-spec] 了解语法和语言特点详情描述



[painless-api-reference]: https://www.elastic.co/guide/en/elasticsearch/painless/7.15/painless-api-reference.html
[modules-scripting-using]: https://www.elastic.co/guide/en/elasticsearch/reference/7.15/modules-scripting-using.html
[painless-lang-spec]: https://www.elastic.co/guide/en/elasticsearch/painless/7.15/painless-lang-spec.html

---
layout: default
title: 重大变化
nav_order: 5
parent: OpenSearch 文档
permalink: /breaking-changes/
---

## 1.x

### 迁移到 OpenSearch 以及对嵌套 JSON 对象数量的限制

当集群包含任何文档，并且所有字段中包含超过 10,000 个嵌套的 JSON 对象时，从 Elasticsearch OSS 版本 6.8 迁移到 OpenSearch 版本 1.x 将失败。Elasticsearch 版本 7.0 引入了该 `index.mapping.nested_objects.limit` 设置来防止内存不足错误，并将该设置指定为默认值 `10000`。OpenSearch 在一开始就采用了此设置，并对嵌套的 JSON 对象实施了限制。但是，由于该设置在 Elasticsearch 6.8 中不存在，并且此版本无法识别该设置，因此迁移到 OpenSearch 1.x 可能会导致不兼容问题，当任何文档中的嵌套 JSON 对象数量超过默认限制时，会阻止 Elasticsearch 6.8 和 OpenSearch 版本 1.x 之间的分片重新定位。

因此，我们建议你在尝试从 Elasticsearch 6.8 迁移之前评估你的数据是否存在这些限制。


## 2.0.0

### 删除映射类型参数

该 `type` 参数已从所有 OpenSearch API 终端节点中删除。相反，索引可以按文档类型进行分类。有关详细信息，请参阅问题[#1940](https://github.com/opensearch-project/opensearch/issues/1940)。

### 弃用非包容性术语

非包容性术语在版本 2.x 中已弃用，并将在 OpenSearch 3.0 中永久删除。我们正在使用以下替代品：

- “白名单”现在是“允许列表”
- “黑名单”现在是“拒绝名单”
- “Master”现在是“Cluster Manager”

### 添加 OpenSearch 通知插件

在 OpenSearch 2.0 中，警报插件现已与用于通知的新插件集成。如果要继续使用 Alerting 插件中的通知操作，请安装新的后端插件 `notifications-core` 和 `notifications`.如果要在 OpenSearch 控制面板中管理通知，请使用新 `notificationsDashboards` 插件。有关更多信息，请参阅[通知]({{site.url}}{{site.baseurl}}/observing-your-data/notifications/index/) OpenSearch 文档页面。

### 删除对 JDK 8 的支持

Lucene 升级迫使 OpenSearch 放弃对 JDK 8 的支持。因此，不再[Java 高级 REST 客户端]({{site.url}}{{site.baseurl}}/clients/java-rest-high-level/)支持 JDK 8. 恢复 JDK 8 支持目前是一项 `opensearch-java` 提案[#156](https://github.com/opensearch-project/opensearch-java/issues/156)，需要从 Java 客户端中删除 OpenSearch 核心作为依赖项（问题[#262](https://github.com/opensearch-project/opensearch-java/issues/262)）。


## 2.5.0

### 文本字段的通配符查询行为

OpenSearch 2.5 包含一个错误修复程序，用于更正文本字段查询参数 `wildcard` 的行为 `case_insensitive`。因此，对忽略区分大小写并在 bug 修复之前错误返回结果的文本字段的通配符查询将不会返回相同的结果。有关更多信息，请参阅 issue [#8711](https://github.com/opensearch-project/OpenSearch/issues/8711)。
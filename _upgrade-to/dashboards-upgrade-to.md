---
layout: default
title: 从 Kibana OSS 迁移到 OpenSearch 控制面板
nav_order: 50
---

# 从 Kibana OSS 迁移到 OpenSearch 控制面板

Kibana OSS 将其可视化和仪表板存储在 Elasticsearch OSS 集群上的一个或多个索引（ `.kibana*`）中。因此，最重要的一步是在从 Elasticsearch OSS 迁移到 OpenSearch 时保持这些索引不变。

请考虑在开始迁移之前导出所有 Kibana 对象。在 Kibana 中，选择**堆栈管理**、**保存的对象**、**导出对象**。{：.tip }

1. 将 Elasticsearch OSS 集群迁移到 OpenSearch 后，请停止 Kibana。

1. 为安全起见，请制作的备份副本 `<kibana-dir>/config/kibana.yml`。

1. 将 OpenSearch 控制面板 tarball 提取到新目录。

1. 将你的设置从 `<kibana-dir>/config/kibana.yml` 移植到 `<dashboards-dir>/config/opensearch_dashboards.yml`。

   通常，名称中带有的设置映射到（例如，和），名称中带有 `elasticsearch` 的设置映射到 `opensearchDashboards` `opensearch`（例如， `elasticsearch.shardTimeout` `kibana.defaultAppId` 和 `opensearch.shardTimeout` `opensearchDashboards.defaultAppId`）。 `kibana` 大多数其他设置使用相同的名称。

   有关 OpenSearch 控制面板设置的完整列表，请参阅[opensearch_dashboards.yml](https://github.com/opensearch-project/OpenSearch-Dashboards/blob/main/config/opensearch_dashboards.yml){：target='\_blank'}。

1. 如果你的 OpenSearch 集群使用安全插件，请保留并修改中的默认设置 `opensearch_dashboards.yml`，特别是 `opensearch.username` 和 `opensearch.password`。

   如果你在 OpenSearch 集群上禁用了安全插件，请删除或注释掉所有 `opensearch_security` 设置。然后运行 `rm -rf plugins/security-dashboards/` 以删除安全插件。

1. 启动 OpenSearch 控制面板：

   ```
   ./bin/opensearch-dashboards
   ```

1. 登录，并验证你保存的搜索、可视化效果和仪表板是否存在。

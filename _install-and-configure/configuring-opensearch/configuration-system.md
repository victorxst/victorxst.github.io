---
layout: default
title: 配置和系统设置
parent: 配置 OpenSearch
nav_order: 10
---

# 配置和系统设置

有关创建 OpenSearch 集群的概述和配置设置示例，请参阅[创建集群]({{site.url}}{{site.baseurl}}/tuning-your-cluster/index/)。要了解有关静态和动态设置的详细信息，请参阅[配置 OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

OpenSearch 支持以下系统设置：

-  `cluster.name`（Static，string）：集群名称。

-  `node.name`（Static，string）：节点的描述性名称。

-  `node.roles`（Static，list）：为 OpenSearch 节点定义一个或多个角色。有效值为 `cluster_manager`、、 `data` `ingest`、、 `search` `ml` 和 `remote_cluster_client` `coordinating_only`。

-  `path.data`（静态，字符串）：存储数据的目录的路径。用逗号分隔多个位置。

-  `path.logs`（Static，string）：日志文件的路径。

-  `bootstrap.memory_lock`（静态，布尔）：在启动时锁定内存。我们建议将堆大小设置为系统上可用内存的一半左右，并允许进程的所有者使用此限制。当系统交换内存时，OpenSearch 的性能不佳。

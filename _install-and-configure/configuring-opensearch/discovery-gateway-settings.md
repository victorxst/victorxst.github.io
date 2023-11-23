---
layout: default
title: 发现和网关设置
parent: 配置 OpenSearch
nav_order: 30
---

# 发现和网关设置

以下是与发现和本地网关相关的设置。

要了解有关静态和动态设置的详细信息，请参阅[配置 OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

## 发现设置

发现过程在形成集群时使用。它包括发现节点和选择集群管理器节点。OpenSearch 支持以下发现设置：

-  `discovery.seed_hosts`（Static，list）：节点启动时执行发现的主机列表。默认主机列表为 `["127.0.0.1", "[::1]"]`。

-  `discovery.seed_providers`（静态，列表）：指定用于获取种子节点地址以启动发现过程的种子主机提供程序的类型。

-  `discovery.type`（静态，字符串）：默认情况下，OpenSearch 形成一个多节点集群。 `single-node` 设置为 `discovery.type` 形成单节点群集。

-  `cluster.initial_cluster_manager_nodes`（Static，list）：用于引导集群的符合集群管理器条件的节点列表。

## 网关设置

本地网关存储集群重启时使用的集群状态和分片数据。OpenSearch 支持以下本地网关设置：

-  `gateway.recover_after_nodes`（静态，整数）：完全群集重新启动后，在恢复开始之前必须加入群集的节点数。

-  `gateway.recover_after_data_nodes`（静态，整数）：完全群集重新启动后，在恢复开始之前必须加入群集的数据节点数。

-  `gateway.expected_data_nodes`（静态，整数）：集群中预期存在的数据节点数。在达到预期数量的节点加入集群后，可以开始恢复本地分片。

-  `gateway.recover_after_time`（静态，时间值）：如果数据节点数小于预期的数据节点数，则在开始恢复之前等待的时间量。

---
layout: default
title: 可用性和恢复设置
parent: 配置 OpenSearch
nav_order: 90
---

# 可用性和恢复设置

可用性和恢复设置包括以下设置：

- [Snapshots](#snapshot-settings)
- [集群管理器任务限制](#cluster-manager-task-throttling-settings)
- [远程支持的存储](#remote-backed-storage-settings)
- [搜索背压](#search-backpressure-settings)
- [分片索引背压](#shard-indexing-backpressure-settings)
- [区段复制](#segment-replication-settings)
- [跨集群复制](#cross-cluster-replication-settings)

要了解有关静态和动态设置的详细信息，请参阅[配置 OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

## 快照设置

OpenSearch 支持以下快照设置：

-  `snapshot.max_concurrent_operations`（动态，整数）：最大并发快照操作数。缺省值为 `1000`。

### 与安全相关的快照设置

有关与安全性相关的快照设置，请参见[安全设置]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/security-settings/)。

### 文件系统设置

有关 Amazon S3 存储库设置的信息，请参阅[Amazon S3]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/snapshots/snapshot-restore/#shared-file-system)。

### Amazon S3 设置

有关 Amazon S3 存储库设置的信息，请参阅[Amazon S3]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/snapshots/snapshot-restore/#amazon-s3)。

## 集群管理器任务限制设置

有关集群管理器任务限制设置的信息，请参阅[设置限制]({{site.url}}{{site.baseurl}}/tuning-your-cluster/cluster-manager-task-throttling/#setting-throttling-limits)。

## 远程支持的存储设置

OpenSearch 支持以下集群级远程支持的存储设置：

-  `cluster.remote_store.translog.buffer_interval`（动态，时间单位）：执行定期 translog 更新时使用的 translog 缓冲区间隔的默认值。仅当索引设置不存在时，此设置 `index.remote_store.translog.buffer_interval` 才有效。

-  `remote_store.moving_average_window_size`（动态，整数）：用于计算通过[远程存储统计信息 API]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/remote-store/remote-store-stats-api/).缺省值为 `20`。强制执行的最小值为 `5`。

有关更多远程支持的存储设置，请参阅[远程支持的存储]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/remote-store/index/)和[配置远程支持的存储]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/remote-store/index/#configuring-remote-backed-storage)。

有关远程管段背压设置，请参见[远程管片背压设置]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/remote-store/remote-segment-backpressure/#remote-segment-backpressure-settings)。

## 搜索背压设置

搜索背压是一种机制，用于识别资源密集型搜索请求，并在节点受到胁迫时取消它们。有关详细信息，请参阅[搜索背压设置]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/search-backpressure/#search-backpressure-settings)。

## 分片索引背压设置

分片索引背压是每个分片级别的智能拒绝机制，可在集群处于压力状态时动态拒绝索引请求。有关更多信息，请参阅分片索引背压[settings]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/shard-indexing-settings/)。

## 分段复制设置

有关分段复制设置的信息，请参见[区段复制]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/segment-replication/index/)。

有关分段复制背压设置的信息，请参见[分段复制背压]({{site.url}}{{site.baseurl}}/tuning-your-cluster/availability-and-recovery/segment-replication/backpressure/)。

## 跨集群复制设置

有关跨集群复制设置的信息，请参见[复制设置]({{site.url}}{{site.baseurl}}/tuning-your-cluster/replication-plugin/settings/)。

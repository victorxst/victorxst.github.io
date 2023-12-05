---
layout: default
title: 快照
nav_order: 5
has_children: true
parent: 可用性和恢复
redirect_from: 
  - /opensearch/snapshots/
  - /opensearch/snapshots/index/
has_toc: false
---

# 快照

快照是集群索引和状态的备份。状态包括集群设置，节点信息，索引元数据（映射，设置或模板）和分配分配。

快照有两个主要用途：

- **从失败中恢复**

  例如，如果簇健康变红，则可以从快照中恢复红色索引。

- **从一个集群迁移到另一个集群**

  例如，如果您要从证明-的-概念是生产集群的概念，您可能会拍摄前者的快照，然后将其恢复后者。


您可以使用[快照API]({{site.url}}{{site.baseurl}}/opensearch/snapshots/snapshot-restore/)。

如果您需要自动化快照创建，则可以使用[快照管理]({{site.url}}{{site.baseurl}}/opensearch/snapshots/snapshot-management/) 特征。


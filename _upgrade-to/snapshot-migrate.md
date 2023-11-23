---
layout: default
title: 使用快照迁移数据
nav_order: 5
---

# 使用快照迁移数据

一种流行的方法是获取[snapshot]({{site.url}}{{site.baseurl}}/opensearch/snapshots/snapshot-restore) Elasticsearch OSS 6.x 或 7.x 索引，在新集群上还原快照，[创建 OpenSearch 集群]({{site.url}}{{site.baseurl}}/opensearch/install/)并将客户端指向新主机。

快照方法可能意味着并行运行两个集群，但允许你在修改 Elasticsearch OSS 集群之前验证 OpenSearch 集群是否以满足你的需求。

---
layout: default
title: 关于迁移过程
nav_order: 1
nav_exclude: true
redirect_from:
  - /upgrade-to/
---

# 关于迁移过程

从 Elasticsearch OSS 迁移到 OpenSearch 的过程因你当前的 Elasticsearch OSS 版本、安装类型、停机容忍度和成本敏感性而异。我们没有采取具体步骤来涵盖所有情况，而是对这一过程有一般指导。

存在三种方法：

- 使用快照连接到[迁移 Elasticsearch OSS 数据]({{site.url}}{{site.baseurl}}/upgrade-to/snapshot-migrate/)新的 OpenSearch 集群。此方法可能会导致停机。
- 在现有节点上执行[重新启动升级或滚动升级]({{site.url}}{{site.baseurl}}/upgrade-to/upgrade-to/)重启升级涉及升级并重启整个集群，而滚动升级需要逐个升级和重启集群中的节点。
- 将现有的 Elasticsearch OSS 节点替换为新的 OpenSearch 节点。节点替换在升级[Docker 集群]({{site.url}}{{site.baseurl}}/upgrade-to/docker-upgrade-to/)时最为常见。

无论采用何种方法，为了防止数据丢失，我们建议你在进行任何迁移之前获取[snapshot]({{site.url}}{{site.baseurl}}/opensearch/snapshots/snapshot-restore)所有索引。

如果你现有的客户端在升级前包含版本检查，例如最新版本的 Logstash OSS 和 Filebeat OSS [检查兼容性]({{site.url}}{{site.baseurl}}/tools/index/#compatibility-matrices)。

有关 OpenSearch 迁移工具的更多信息，请参阅[OpenSearch 升级、迁移和比较工具]({{site.url}}{{site.baseurl}}/tools/index/#opensearch-upgrade-migration-and-comparison-tools)。

## 从 Open Distro 升级

有关从 Open Distro 升级到 OpenSearch 的步骤，请参阅博客文章[操作方法：从 Open Distro 升级到 OpenSearch](https://opensearch.org/blog/technical-posts/2021/07/how-to-upgrade-from-opendistro-to-opensearch/)。
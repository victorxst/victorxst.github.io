---
layout: default
title: 将 Docker 集群迁移到 OpenSearch
nav_order: 25
---

# 将 Docker 集群迁移到 OpenSearch

如果你使用 Kubernetes 等容器编排系统（或手动管理容器）并希望避免停机，请不要将该过程视为每个节点的升级，而是每个节点的停用和替换。将 OpenSearch 节点逐个添加到集群并删除 Elasticsearch OSS 节点，根据需要指向现有数据卷，并留出时间让所有索引恢复到绿色状态，然后再继续。

如果你使用 Docker Compose，我们强烈建议你执行相当于[集群重启升级]({{site.url}}{{site.baseurl}}/upgrade-to/upgrade-to/).使用新映像、新设置和新环境变量更新群集配置，并对其进行测试。然后停止并启动群集。此过程需要停机，但只需很少的步骤，并允许你继续将群集视为可以可靠地部署和重新部署的单个实体。

最重要的一步是保持数据量完好无损。**不要**运行 `docker-compose down -v`。

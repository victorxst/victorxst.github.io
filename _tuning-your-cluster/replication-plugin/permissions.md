---
layout: default
title: 复制安全性
nav_order: 30
parent: 跨集群复制
redirect_from:
  - /replication-plugin/permissions/
---

# 跨集群复制安全性

您可以使用[安全插件]({{site.url}}{{site.baseurl}}/security/index/) 与十字架-集群复制以将用户限制为某些操作。例如，您可能希望某些用户仅对领导者或追随者集群执行复制活动。

因为交叉-群集复制涉及多个群集，群集可能具有不同的安全配置。支持以下配置：

- 安全插件在两个群集上完全启用
- 安全插件仅针对两个群集上的TLS启用（`plugins.security.ssl_only`）
- 两个群集上没有安全插件或禁用（不建议）

启用节点-到-对领导者和追随者群集上的节点加密，以确保簇之间的复制流量加密。

## 基本权限

为了非-管理用户要执行复制活动，必须将其映射到适当的权限。

安全插件有两个构建-在涵盖大多数复制用例的角色中：`cross_cluster_replication_leader_full_access`，它为领导者集群提供了复制权限，并且`cross_cluster_replication_follower_full_access`，它在追随者群集上提供了复制权限。对于每个描述，请参阅[预定义的角色]({{site.url}}{{site.baseurl}}/security/access-control/users-roles#predefined-roles)。

如果您不想使用默认角色，则可以组合单个复制[权限]({{site.url}}{{site.baseurl}}/tuning-your-cluster/replication-plugin/permissions/#replication-permissions) 满足您的需求。大多数权限对应于特定的REST API操作。例如，`indices:admin/plugins/replication/index/pause` 许可使您可以暂停复制。

## 映射领导者和追随者集群角色

这[开始复制]({{site.url}}{{site.baseurl}}/replication-plugin/api/#start-replication) 和[创建复制规则]({{site.url}}{{site.baseurl}}/replication-plugin/api/#create-replication-rule) 操作是特殊情况。它们涉及必须与角色相关联的领导者和追随者群集的背景过程。当您执行这些操作之一时，您必须明确通过`leader_cluster_role` 和
`follower_cluster_role` 在请求中，OpenSearch然后在所有后端复制任务中使用。

启用非-管理员开始复制并创建复制规则，在每个群集上创建一个相同的用户（例如，`replication_user`）并将其映射到`cross_cluster_replication_leader_full_access` 在遥控集群中的角色，`cross_cluster_replication_follower_full_access` 在追随者集群上。有关说明，请参阅[将用户映射到角色]({{site.url}}{{site.baseurl}}/security/access-control/users-roles/#map-users-to-roles)。

然后将这些角色添加到请求中，并用适当的凭据签名：

```bash
curl -XPUT -k -H 'Content-Type: application/json' -u 'replication_user:password' 'https://localhost:9200/_plugins/_replication/follower-01/_start?pretty' -d '
{
   "leader_alias": "leader-cluster",
   "leader_index": "leader-01",
   "use_roles":{
      "leader_cluster_role": "cross_cluster_replication_leader_full_access",
      "follower_cluster_role": "cross_cluster_replication_follower_full_access"
   }
}'
```

您可以使用各个权限创建自己的自定义领导者和追随者集群角色，但我们建议使用默认角色，这非常适合大多数用例。

## 复制许可

以下各节列出了可用索引和群集-交叉的水平许可-集群复制。

### 追随者集群

安全插件支持这些追随者群集的这些权限：

```
indices:admin/plugins/replication/index/setup/validate
indices:admin/plugins/replication/index/start
indices:admin/plugins/replication/index/pause
indices:admin/plugins/replication/index/resume
indices:admin/plugins/replication/index/stop
indices:admin/plugins/replication/index/update
indices:admin/plugins/replication/index/status_check
indices:data/write/plugins/replication/changes
cluster:admin/plugins/replication/autofollow/update
```

### 领导者集群

安全插件支持领导者集群的这些权限：

```
indices:admin/plugins/replication/validate
indices:data/read/plugins/replication/file_chunk
indices:data/read/plugins/replication/changes
```


---
layout: default
title: 自动关注
nav_order: 20
parent: 跨集群复制
redirect_from:
  - /replication-plugin/auto-follow/
---

# 自动关注跨集群复制

自动关注您可以根据匹配模式自动复制在领导者群集上创建的索引。当您在领导者群集上创建索引时，其名称与指定模式匹配（例如，`index-01*`），在追随者群集上自动创建相应的追随者索引。

您可以为一个群集配置多个复制规则。这些模式当前仅支持通配符匹配。

## 先决条件

你需要[建立十字架-集群连接]({{site.url}}{{site.baseurl}}/replication-plugin/get-started/#set-up-a-cross-cluster-connection) 在启用自动之前的两个群集之间-跟随。

## 权限

如果启用了安全插件，请确保-管理用户被映射到适当的权限，以便他们可以执行复制操作。对于索引和群集-级别许可要求，请参阅[叉-集群复制权限]({{site.url}}{{site.baseurl}}/replication-plugin/permissions/)。

## 开始使用汽车-跟随

复制规则是您针对单个追随者群集创建的模式的集合。创建复制规则时，它首先是自动复制任何与模式匹配的 *现有 *索引。然后，它将继续复制您创建与模式相匹配的任何 *新 *索引。

在追随者群集上创建复制规则：

```bash
curl -XPOST -k -H 'Content-Type: application/json' -u 'admin:admin' 'https://localhost:9200/_plugins/_replication/_autofollow?pretty' -d '
{
   "leader_alias" : "my-connection-alias",
   "name": "my-replication-rule",
   "pattern": "movies*",
   "use_roles":{
      "leader_cluster_role": "all_access",
      "follower_cluster_role": "all_access"
   }
}'
```

如果禁用了安全插件，则可以忽略`use_roles` 范围。但是，如果启用了它，则需要指定OpenSearch用来身份验证请求的领导者和追随者集群角色。此示例使用`all_access` 为简单起见，但是我们建议在每个群集上创建复制用户，并且[相应地绘制它]({{site.url}}{{site.baseurl}}/replication-plugin/permissions/#map-the-leader-and-follower-cluster-roles)。
{: .tip }

要测试该规则，请在领导者集群上创建匹配索引：

```bash
curl -XPUT -k -H 'Content-Type: application/json' -u 'admin:admin' 'https://localhost:9201/movies-0001?pretty'
```

并确认其副本显示在追随者集群上：

```bash
curl -XGET -u 'admin:admin' -k 'https://localhost:9200/_cat/indices?v'
```

索引出现可能需要几秒钟。

```bash
health status index        uuid                     pri rep docs.count docs.deleted store.size pri.store.size
yellow open   movies-0001  kHOxYYHxRMeszLjTD9rvSQ     1   1          0            0       208b           208b
```

## 检索复制规则

要检索在集群上配置的现有复制规则的列表，请发送以下请求：

```bash
curl -XGET -u 'admin:admin' -k 'https://localhost:9200/_plugins/_replication/autofollow_stats'

{
   "num_success_start_replication": 1,
   "num_failed_start_replication": 0,
   "num_failed_leader_calls": 0,
   "failed_indices":[
      
   ],
   "autofollow_stats":[
      {
         "name":"my-replication-rule",
         "pattern":"movies*",
         "num_success_start_replication": 1,
         "num_failed_start_replication": 0,
         "num_failed_leader_calls": 0,
         "failed_indices":[
            
         ]
      }
   ]
}
```

## 删除复制规则

要删除复制规则，请将以下请求发送到追随者集群：

```bash
curl -XDELETE -k -H 'Content-Type: application/json' -u 'admin:admin' 'https://localhost:9200/_plugins/_replication/_autofollow?pretty' -d '
{
   "leader_alias" : "my-conection-alias",
   "name": "my-replication-rule"
}'
```

当您删除复制规则时，OpenSearch停止复制 *新 *符合模式的索引，但是现有的索引是先前创建的规则保留的索引-仅并继续复制。如果您需要停止现有复制活动并打开索引以进行写入，请使用[停止复制API操作]({{site.url}}{{site.baseurl}}/replication-plugin/api/#stop-replication)


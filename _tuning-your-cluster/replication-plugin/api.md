---
layout: default
title: API
nav_order: 50
parent: 跨集群复制
redirect_from:
  - /replication-plugin/api/
---

# 跨集群复制API

使用这些复制操作来编程管理跨集群复制。

#### 目录
- TOC
{：toc}

## 开始复制
引入了1.1
{：.label .label-紫色的 }

启动从领导者群集到追随者群集的索引复制。将此请求发送到追随者集群。


#### 要求

```json
PUT /_plugins/_replication/<follower-index>/_start
{
   "leader_alias":"<connection-alias-name>",
   "leader_index":"<index-name>",
   "use_roles":{
      "leader_cluster_role":"<role-name>",
      "follower_cluster_role":"<role-name>"
   }
}
```

指定以下选项：

选项| 描述| 类型| 必需的
：--- | ：--- |：--- |：--- |
`leader_alias` |  十字架的名字-集群连接。您定义这个别名[建立十字架-集群连接]({{site.url}}{{site.baseurl}}/replication-plugin/get-started/#set-up-a-cross-cluster-connection)。| `string` | 是的
`leader_index` |  您要复制的领导者集群的索引。| `string` | 是的
`use_roles` |  索引之间的所有后续后端复制任务要使用的角色。指定`leader_cluster_role` 和`follower_cluster_role`。看[映射领导者和追随者集群角色]({{site.url}}{{site.baseurl}}/replication-plugin/permissions/#map-the-leader-and-follower-cluster-roles)。| `string` | 如果启用了安全插件

#### 示例响应

```json
{
   "acknowledged": true
}
```

## 停止复制
引入了1.1
{：.label .label-紫色的 }

终止复制并将追随者索引转换为标准索引。将此请求发送到追随者集群。

#### 要求

```json
POST /_plugins/_replication/<follower-index>/_stop
{}
```

#### 示例响应

```json
{
   "acknowledged": true
}
```

## 暂停复制
引入了1.1
{：.label .label-紫色的 }

暂停领导索引的复制。将此请求发送到追随者集群。

#### 要求

```json
POST /_plugins/_replication/<follower-index>/_pause 
{}
```

在暂停超过12个小时后，您无法恢复复制。你必须[停止复制]({{site.url}}{{site.baseurl}}/replication-plugin/api/#stop-replication)，删除追随者索引，然后重新启动领导者的复制。

#### 示例响应

```json
{
   "acknowledged": true
}
```

## 恢复复制
引入了1.1
{：.label .label-紫色的 }

恢复领导索引的复制。将此请求发送到追随者集群。

#### 要求

```json
POST /_plugins/_replication/<follower-index>/_resume
{}
```

#### 示例响应

```json
{
   "acknowledged": true
}
```

## 获取复制状态
引入了1.1
{：.label .label-紫色的 }

获取索引复制的状态。可能的状态是`SYNCING`，，，，`BOOTSTRAPING`，，，，`PAUSED`， 和`REPLICATION NOT IN PROGRESS`。使用同步详细信息来测量复制滞后。将此请求发送到追随者集群。

#### 要求

```json
GET /_plugins/_replication/<follower-index>/_status
```

#### 示例响应

```json
{
  "status" : "SYNCING",
  "reason" : "User initiated",
  "leader_alias" : "my-connection-name",
  "leader_index" : "leader-01",
  "follower_index" : "follower-01",
  "syncing_details" : {
    "leader_checkpoint" : 19,
    "follower_checkpoint" : 19,
    "seq_no" : 0
  }
}
```
要在响应中包含碎片复制细节，请添加`&verbose=true` 范围。

领导者和追随者检查点值以负整数开头，并反映碎片计数（-1碎片，-5对于五片，依此类推）。随着您所做的每次更改，这些值都会向积极整数增长。例如，当您对领导索引进行更改时`leader_checkpoint` 变成`0`。这`follower_checkpoint` 最初静止`-1` 直到追随者索引从领导者那里拉动更改为止，此时它会增加`0`。如果值相同，则表示索引已完全同步。

## 获取领导者群集统计
引入了1.1
{：.label .label-紫色的 }

获取有关指定群集上复制领导者索引的信息。

#### 要求

```json
GET /_plugins/_replication/leader_stats
```

#### 示例响应

```json
{
   "num_replicated_indices": 2,
   "operations_read": 15,
   "translog_size_bytes": 1355,
   "operations_read_lucene": 0,
   "operations_read_translog": 15,
   "total_read_time_lucene_millis": 0,
   "total_read_time_translog_millis": 659,
   "bytes_read": 1000,
   "index_stats":{
      "leader-index-1":{
         "operations_read": 7,
         "translog_size_bytes": 639,
         "operations_read_lucene": 0,
         "operations_read_translog": 7,
         "total_read_time_lucene_millis": 0,
         "total_read_time_translog_millis": 353,
         "bytes_read":466
      },
      "leader-index-2":{
         "operations_read": 8,
         "translog_size_bytes": 716,
         "operations_read_lucene": 0,
         "operations_read_translog": 8,
         "total_read_time_lucene_millis": 0,
         "total_read_time_translog_millis": 306,
         "bytes_read": 534
      }
   }
}
```

## 获取追随者群集统计
引入了1.1
{：.label .label-紫色的 }

获取有关指定群集上的追随者（同步）索引的信息。

#### 要求

```json
GET /_plugins/_replication/follower_stats
```

#### 示例响应

```json
{
   "num_syncing_indices": 2,
   "num_bootstrapping_indices": 0,
   "num_paused_indices": 0,
   "num_failed_indices": 0,
   "num_shard_tasks": 2,
   "num_index_tasks": 2,
   "operations_written": 3,
   "operations_read": 3,
   "failed_read_requests": 0,
   "throttled_read_requests": 0,
   "failed_write_requests": 0,
   "throttled_write_requests": 0,
   "follower_checkpoint": 1,
   "leader_checkpoint": 1,
   "total_write_time_millis": 2290,
   "index_stats":{
      "follower-index-1":{
         "operations_written": 2,
         "operations_read": 2,
         "failed_read_requests": 0,
         "throttled_read_requests": 0,
         "failed_write_requests": 0,
         "throttled_write_requests": 0,
         "follower_checkpoint": 1,
         "leader_checkpoint": 1,
         "total_write_time_millis": 1355
      },
      "follower-index-2":{
         "operations_written": 1,
         "operations_read": 1,
         "failed_read_requests": 0,
         "throttled_read_requests": 0,
         "failed_write_requests": 0,
         "throttled_write_requests": 0,
         "follower_checkpoint": 0,
         "leader_checkpoint": 0,
         "total_write_time_millis": 935
      }
   }
}
```

## 获取自动-遵循统计数据
引入了1.1
{：.label .label-紫色的 }

获取有关汽车的信息-关注活动和在指定集群上配置的任何复制规则。

#### 要求

```json
GET /_plugins/_replication/autofollow_stats
```

#### 示例响应

```json
{
   "num_success_start_replication": 2,
   "num_failed_start_replication": 0,
   "num_failed_leader_calls": 0,
   "failed_indices":[
      
   ],
   "autofollow_stats":[
      {
         "name":"my-replication-rule",
         "pattern":"movies*",
         "num_success_start_replication": 2,
         "num_failed_start_replication": 0,
         "num_failed_leader_calls": 0,
         "failed_indices":[
            
         ]
      }
   ]
}
```

## 更新设置
引入了1.1
{：.label .label-紫色的 }

更新追随者索引上的设置。

#### 要求

```json
PUT /_plugins/_replication/<follower-index>/_update
{
   "settings":{
      "index.number_of_shards": 4,
      "index.number_of_replicas": 2
   }
}
```

#### 示例响应

```json
{
   "acknowledged": true
}
```

## 创建复制规则
引入了1.1
{：.label .label-紫色的 }

自动开始复制与指定模式匹配的索引。如果领导者群集上的新索引与模式匹配，则OpenSearch会自动创建关注者索引并开始复制。您也可以使用此API更新现有的复制规则。

将此请求发送到追随者集群。

确保注意所有自动的名称-创建它们后，请遵循模式。当前，复制插件不包括API操作以检索现有模式列表。
{： 。提示 }

#### 要求

```json
POST /_plugins/_replication/_autofollow
{
   "leader_alias" : "<connection-alias-name>",
   "name": "<auto-follow-pattern-name>",
   "pattern": "<pattern>",
   "use_roles":{
      "leader_cluster_role": "<role-name>",
      "follower_cluster_role": "<role-name>"
   }
}
```

指定以下选项：

选项| 描述| 类型| 必需的
：--- | ：--- |：--- |：--- |
`leader_alias` |  十字架的名字-集群连接。您定义这个别名[建立十字架-集群连接]({{site.url}}{{site.baseurl}}/replication-plugin/get-started/#set-up-a-cross-cluster-connection)。| `string` | 是的
`name` |  汽车的名字-遵循模式。| `string` | 是的
`pattern` |  在指定的领导者群集中与索引相匹配的一系列索引模式。支持通配符角色。例如，`leader-*`。| `string` | 是的
`use_roles` |  索引之间的所有后续后端复制任务要使用的角色。指定`leader_cluster_role` 和`follower_cluster_role`。看[映射领导者和追随者集群角色]({{site.url}}{{site.baseurl}}/replication-plugin/permissions/#map-the-leader-and-follower-cluster-roles)。| `string` | 如果启用了安全插件

#### 示例响应

```json
{
   "acknowledged": true
}
```

## 删除复制规则
引入了1.1
{：.label .label-紫色的 }

删除指定的复制规则。此操作防止任何新索引被复制，但不会停止该规则已经启动的现有复制。复制的索引仍然阅读-直到您停止复制为止。

将此请求发送到追随者集群。

#### 要求

```json
DELETE /_plugins/_replication/_autofollow
{
   "leader_alias" : "<connection-alias-name>",
   "name": "<auto-follow-pattern-name>",
}
```

指定以下选项：

选项| 描述| 类型| 必需的
：--- | ：--- |：--- |：--- |
`leader_alias` |  十字架的名字-集群连接。您定义这个别名[建立十字架-集群连接]({{site.url}}{{site.baseurl}}/replication-plugin/get-started/#set-up-a-cross-cluster-connection)。| `string` | 是的
`name` |  图案的名称。| `string` | 是的

#### 示例响应

```json
{
   "acknowledged": true
}
```


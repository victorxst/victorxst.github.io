---
layout: default
title: 群集分配解释
nav_order: 10
parent: 群集API
has_children: false
redirect_from:
 - /opensearch/rest-api/cluster-allocation/
---

# 群集分配解释
**引入1.0**
{: .label .label-purple }

最基本的群集分配解释请求找到了一个未分配的碎片，并解释了为什么不能将其分配给节点。

如果添加了一些选项，则可以在特定的碎片上获取信息，包括为什么OpenSearch将其分配给当前节点。


## 例子

```json
GET _cluster/allocation/explain?include_yes_decisions=true
{
  "index": "movies",
  "shard": 0,
  "primary": true
}
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET _cluster/allocation/explain
POST _cluster/allocation/explain
```


## URL参数

所有群集分配解释参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
包括_yes_decisions| 布尔| 当试图将碎片分配给节点时，OpenSearch做出了一系列的决定。如果此参数为真，则OpenSearch包括（通常更多）"yes" 在其回应中的决定。默认值为false。
包括_disk_info| 布尔| 是否在响应中包括有关磁盘使用情况的信息。默认值为false。


## 请求身体

所有群集分配解释字段都是可选的。

场地| 类型| 描述
:--- | :--- | :---
Current_Node| 细绳| 如果您只想说明碎片恰好在特定节点上，请在此处指定该节点名称。
指数| 细绳| 碎片索引的名称。
基本的| 布尔| 是否提供对主要碎片（true）或其第一个副本（false）的解释，该副本（false）共享相同的碎片ID。
碎片| 整数| 您想要解释的碎片ID。


## 回复

```json
{
  "index": "movies",
  "shard": 0,
  "primary": true,
  "current_state": "started",
  "current_node": {
    "id": "d8jRZcW1QmCBeVFlgOJx5A",
    "name": "opensearch-node1",
    "transport_address": "172.24.0.4:9300",
    "weight_ranking": 1
  },
  "can_remain_on_current_node": "yes",
  "can_rebalance_cluster": "yes",
  "can_rebalance_to_other_node": "no",
  "rebalance_explanation": "cannot rebalance as no target node exists that can both allocate this shard and improve the cluster balance",
  "node_allocation_decisions": [{
    "node_id": "vRxi4uPcRt2BtHlFoyCyTQ",
    "node_name": "opensearch-node2",
    "transport_address": "172.24.0.3:9300",
    "node_decision": "no",
    "weight_ranking": 1,
    "deciders": [{
        "decider": "max_retry",
        "decision": "YES",
        "explanation": "shard has no previous failures"
      },
      {
        "decider": "replica_after_primary_active",
        "decision": "YES",
        "explanation": "shard is primary and can be allocated"
      },
      {
        "decider": "enable",
        "decision": "YES",
        "explanation": "all allocations are allowed"
      },
      {
        "decider": "node_version",
        "decision": "YES",
        "explanation": "can relocate primary shard from a node with version [1.0.0] to a node with equal-or-newer version [1.0.0]"
      },
      {
        "decider": "snapshot_in_progress",
        "decision": "YES",
        "explanation": "no snapshots are currently running"
      },
      {
        "decider": "restore_in_progress",
        "decision": "YES",
        "explanation": "ignored as shard is not being recovered from a snapshot"
      },
      {
        "decider": "filter",
        "decision": "YES",
        "explanation": "node passes include/exclude/require filters"
      },
      {
        "decider": "same_shard",
        "decision": "NO",
        "explanation": "a copy of this shard is already allocated to this node [[movies][0], node[vRxi4uPcRt2BtHlFoyCyTQ], [R], s[STARTED], a[id=x8w7QxWdQQa188HKGn0iMQ]]"
      },
      {
        "decider": "disk_threshold",
        "decision": "YES",
        "explanation": "enough disk for shard on node, free: [35.9gb], shard size: [15.1kb], free after allocating shard: [35.9gb]"
      },
      {
        "decider": "throttling",
        "decision": "YES",
        "explanation": "below shard recovery limit of outgoing: [0 < 2] incoming: [0 < 2]"
      },
      {
        "decider": "shards_limit",
        "decision": "YES",
        "explanation": "total shard limits are disabled: [index: -1, cluster: -1] <= 0"
      },
      {
        "decider": "awareness",
        "decision": "YES",
        "explanation": "allocation awareness is not enabled, set cluster setting [cluster.routing.allocation.awareness.attributes] to enable it"
      }
    ]
  }]
}
```


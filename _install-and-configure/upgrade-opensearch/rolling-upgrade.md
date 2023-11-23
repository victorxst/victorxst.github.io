---
layout: default
title: 滚动升级
parent: 升级 OpenSearch
nav_order: 10
---

# 滚动升级

滚动升级（有时称为“节点替换升级”）可以在正在运行的集群上执行，几乎无需停机。节点单独停止并就地升级。或者，可以由运行新版本的主机一次停止和替换一个节点。在此过程中，你可以继续为集群中的数据编制索引和查询。

本文档是滚动升级过程的高级、与平台无关的概述。有关命令、脚本和配置文件的具体示例，请参阅[Appendix]({{site.url}}{{site.baseurl}}/upgrade-opensearch/appendix/)。

## 准备升级

在对 OpenSearch 集群进行任何更改之前，请查看[升级 OpenSearch]({{site.url}}{{site.baseurl}}/upgrade-opensearch/index/)有关备份配置文件以及创建集群状态和索引快照的建议。

**重要：** OpenSearch 节点无法降级。如果你需要恢复升级，则需要执行 OpenSearch 的全新安装并从快照还原集群。在开始升级过程之前，拍摄快照并将其存储在远程存储库中。{：.important}

## 执行升级

1. 在开始之前，请验证 OpenSearch 集群的运行状况。在升级之前，应解决任何索引或分片分配问题，以确保保留数据。状态**绿**表示已分配所有主分片和副本分片。有关详细信息，请参阅[群集运行状况]({{site.url}}{{site.baseurl}}/api-reference/cluster-api/cluster-health/)。以下命令查询 `_cluster/health` API 端点：
   ```json
   GET "/_cluster/health?pretty"
   ```
   响应应类似于以下示例：The response should look similar to the following example：
   ```json
   {
       "cluster_name":"opensearch-dev-cluster",
       "status":"green",
       "timed_out":false,
       "number_of_nodes":4,
       "number_of_data_nodes":4,
       "active_primary_shards":1,
       "active_shards":4,
       "relocating_shards":0,
       "initializing_shards":0,
       "unassigned_shards":0,
       "delayed_unassigned_shards":0,
       "number_of_pending_tasks":0,
       "number_of_in_flight_fetch":0,
       "task_max_waiting_in_queue_millis":0,
       "active_shards_percent_as_number":100.0
   }
   ```
1. 禁用分片复制以防止在节点脱机时创建分片副本。这将停止集群中节点上 Lucene 索引段的移动。你可以通过查询 `_cluster/settings` API 终端节点来禁用分片复制：
   ```json
   PUT "/_cluster/settings?pretty"
   {
       "persistent": {
           "cluster.routing.allocation.enable": "primaries"
       }
   }
   ```
   响应应类似于以下示例：The response should look similar to the following example：
   ```json
   {
     "acknowledged" : true,
     "persistent" : {
       "cluster" : {
         "routing" : {
           "allocation" : {
             "enable" : "primaries"
           }
         }
       }
     },
     "transient" : { }
   }
   ```

1. 在群集上执行刷新操作，以将事务日志条目提交到 Lucene 索引：
   ```json
   POST "/_flush?pretty"
   ```
   响应应类似于以下示例：The response should look similar to the following example：
   ```json
   {
     "_shards" : {
       "total" : 4,
       "successful" : 4,
       "failed" : 0
     }
   }
   ```
1. 查看群集并确定要升级的第一个节点。符合条件的集群管理器节点应最后升级，因为 OpenSearch 节点可以加入运行较旧版本的管理器节点的集群，但不能加入运行较新版本的所有管理器节点的集群。
1. 查询 `_cat/nodes` 端点以确定哪个节点已提升为集群管理器。以下命令包含仅请求 name、version、node.role 和 master 标头的其他查询参数。请注意，OpenSearch 1.x 版本使用术语“master”，该术语在 OpenSearch 2.x 及更高版本中已被弃用并替换为“cluster_manager”。
   ```bash
   GET "/_cat/nodes?v&h=name,version,node.role,master" | column -t
   ```
   响应应类似于以下示例：The response should look similar to the following example：
   ```bash
   name        version  node.role  master
   os-node-01  7.10.2   dimr       -
   os-node-04  7.10.2   dimr       -
   os-node-03  7.10.2   dimr       -
   os-node-02  7.10.2   dimr       *
   ```
1. 停止要升级的节点。删除容器时，请勿删除与容器关联的卷。新的 OpenSearch 容器将使用现有卷。**删除卷将导致数据丢失**。
1. 通过查询 `_cat/nodes` API 端点，确认关联的节点已从集群中移除：
   ```bash
   GET "/_cat/nodes?v&h=name,version,node.role,master" | column -t
   ```
   响应应类似于以下示例：The response should look similar to the following example：
   ```bash
   name        version  node.role  master
   os-node-02  7.10.2   dimr       *
   os-node-04  7.10.2   dimr       -
   os-node-03  7.10.2   dimr       -
   ```
    `os-node-01` 不再列出，因为容器已被停止和删除。
1. 部署运行所需版本的 OpenSearch 的新容器，并将其映射到与你删除的容器相同的卷。
1. 在 OpenSearch 在新节点上运行后查询 `_cat/nodes` 终端节点，以确认它已加入集群：
   ```bash
   GET "/_cat/nodes?v&h=name,version,node.role,master" | column -t
   ```
   响应应类似于以下示例：The response should look similar to the following example：
   ```bash
   name        version  node.role  master
   os-node-02  7.10.2   dimr       *
   os-node-04  7.10.2   dimr       -
   os-node-01  7.10.2   dimr       -
   os-node-03  7.10.2   dimr       -
   ```
   在示例输出中，新的 OpenSearch 节点向集群报告正在运行的 `7.10.2` 版本。这是 `compatibility.override_main_response_version` 的结果，在连接到具有检查版本的旧客户端的集群时使用。你可以通过调用 `/_nodes` API 端点来手动确认节点的版本，如以下命令所示。替换为 `<nodeName>` 节点的名称。请参阅[Nodes API]({{site.url}}{{site.baseurl}}/api-reference/nodes-apis/index/)以了解更多信息。
   ```bash
   GET "/_nodes/<nodeName>?pretty=true" | jq -r '.nodes | .[] | "\(.name) v\(.version)"'
   ```
   响应应类似于以下示例：The response should look similar to the following example：
   ```bash
   os-node-01 v1.3.7
   ```
1. 对群集中的每个节点重复步骤 5 到 9. 请记住最后升级符合条件的集群管理器节点。替换最后一个节点后，查询 `_cat/nodes` 终端节点以确认所有节点都已加入群集。集群现在已引导到新版本的 OpenSearch。你可以通过查询 `_cat/nodes` API 终端节点来验证集群版本：
   ```bash
   GET "/_cat/nodes?v&h=name,version,node.role,master" | column -t
   ```
   响应应类似于以下示例：The response should look similar to the following example：
   ```bash
   name        version  node.role  master
   os-node-04  1.3.7    dimr       -
   os-node-02  1.3.7    dimr       *
   os-node-01  1.3.7    dimr       -
   os-node-03  1.3.7    dimr       -
   ```
1. 重新启用分片复制：
   ```json
   PUT "/_cluster/settings?pretty"
   {
       "persistent": {
           "cluster.routing.allocation.enable": "all"
       }
   }
   ```
   响应应类似于以下示例：The response should look similar to the following example：
   ```json
   {
     "acknowledged" : true,
     "persistent" : {
       "cluster" : {
         "routing" : {
           "allocation" : {
             "enable" : "all"
           }
         }
       }
     },
     "transient" : { }
   }
   ```
1. 确认群集运行状况良好：
   ```bash
   GET "/_cluster/health?pretty"
   ```
   响应应类似于以下示例：The response should look similar to the following example：
   ```json
   {
     "cluster_name" : "opensearch-dev-cluster",
     "status" : "green",
     "timed_out" : false,
     "number_of_nodes" : 4,
     "number_of_data_nodes" : 4,
     "discovered_master" : true,
     "active_primary_shards" : 1,
     "active_shards" : 4,
     "relocating_shards" : 0,
     "initializing_shards" : 0,
     "unassigned_shards" : 0,
     "delayed_unassigned_shards" : 0,
     "number_of_pending_tasks" : 0,
     "number_of_in_flight_fetch" : 0,
     "task_max_waiting_in_queue_millis" : 0,
     "active_shards_percent_as_number" : 100.0
   }
   ```
1. 升级现已完成，你可以开始享受最新功能和修复！

### 相关文章

- [OpenSearch 配置]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/)
- [性能分析器]({{site.url}}{{site.baseurl}}/monitoring-plugins/pa/index/)
- [安装和配置 OpenSearch 控制面板]({{site.url}}{{site.baseurl}}/install-and-configure/install-dashboards/index/)
- [关于 OpenSearch 中的安全性]({{site.url}}{{site.baseurl}}/security/index/)

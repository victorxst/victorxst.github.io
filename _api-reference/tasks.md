---
layout: default
title: 任务
nav_order: 85
redirect_from:
 - /opensearch/rest-api/tasks/
---

# 任务
**引入1.0**
{: .label .label-purple }

任务是您在集群中运行的任何操作。例如，搜索书籍的数据收集以获取标题或作者名称是一项任务。运行OpenSearch时，将自动创建任务以监视群集的健康和性能。有关当前集群中当前执行的所有任务的更多信息，您可以使用`tasks` API操作。

以下请求返回有关您所有任务的信息：

```
GET _tasks
```
{% include copy-curl.html %}

通过包含任务ID，您可以获取特定于特定任务的信息。请注意，任务ID由节点的标识字符串和任务的数值ID组成。例如，如果您的节点的识别字符串为`nodestring` 任务的数值ID是`1234`，那么您的任务ID是`nodestring:1234`。您可以通过运行`tasks` 手术：

```
GET _tasks/<task_id>
```
{% include copy-curl.html %}

请注意，如果任务完成了，则不会作为您的请求的一部分返回。对于一个任务需要更长的时间才能完成的示例，您可以运行[`_reindex`]({{site.url}}{{site.baseurl}}/opensearch/reindex-data) API在较大的文档上操作，然后运行`tasks`。

#### 示例响应
```json
{
  "nodes": {
    "Mgqdm0r9SEGClWxp_RbnaQ": {
      "name": "opensearch-node1",
      "transport_address": "172.18.0.3:9300",
      "host": "172.18.0.3",
      "ip": "172.18.0.3:9300",
      "roles": [
        "data",
        "ingest",
        "master",
        "remote_cluster_client"
      ],
      "tasks": {
        "Mgqdm0r9SEGClWxp_RbnaQ:17416": {
          "node": "Mgqdm0r9SEGClWxp_RbnaQ",
          "id": 17416,
          "type": "transport",
          "action": "cluster:monitor/tasks/lists",
          "start_time_in_millis": 1613599752458,
          "running_time_in_nanos": 994000,
          "cancellable": false,
          "headers": {}
          }
        },
        "Mgqdm0r9SEGClWxp_RbnaQ:17413": {
          "node": "Mgqdm0r9SEGClWxp_RbnaQ",
          "id": 17413,
          "type": "transport",
          "action": "indices:data/write/bulk",
          "start_time_in_millis": 1613599752286,
          "running_time_in_nanos": 172846500,
          "cancellable": false,
          "parent_task_id": "Mgqdm0r9SEGClWxp_RbnaQ:17366",
          "headers": {}
        },
        "Mgqdm0r9SEGClWxp_RbnaQ:17366": {
          "node": "Mgqdm0r9SEGClWxp_RbnaQ",
          "id": 17366,
          "type": "transport",
          "action": "indices:data/write/reindex",
          "start_time_in_millis": 1613599750929,
          "running_time_in_nanos": 1529733100,
          "cancellable": true,
          "headers": {}
        }
      }
    }
  }
}
```

您也可以在查询中使用以下参数。

范围| 数据类型| 描述|
:--- | :--- | :---
`nodes` | 列表| 逗号-分开的节点ID列表或名称以限制返回的信息。使用`_local` 要返回您要连接的节点的信息，请指定节点名称以从特定节点获取信息，或将参数保留为空以从所有节点中获取信息。
`actions` | 列表| 逗号-分开应返回的动作列表。保持空返回全部。
`detailed` | 布尔| 返回详细的任务信息。（默认：false）
`parent_task_id` | 细绳| 使用指定的父任务ID（node_id：task_number）返回任务。保持空或设置为-1返回全部。
`wait_for_completion` | 布尔| 等待匹配任务完成。（默认：false）
`group_by` | 枚举| 按父母/子关系或节点组成任务。（默认：节点）
`timeout` | 时间| 明确的操作超时。（默认：30秒）
`cluster_manager_timeout` | 时间| 等待与主节点连接的时间。（默认：30秒）

例如，此请求返回当前在名为的节点上运行的任务`opensearch-node1`：

#### 示例请求

```json
GET /_tasks?nodes=opensearch-node1
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "nodes": {
    "Mgqdm0r9SEGClWxp_RbnaQ": {
      "name": "opensearch-node1",
      "transport_address": "sample_address",
      "host": "sample_host",
      "ip": "sample_ip",
      "roles": [
        "data",
        "ingest",
        "master",
        "remote_cluster_client"
      ],
      "tasks": {
        "Mgqdm0r9SEGClWxp_RbnaQ:24578": {
          "node": "Mgqdm0r9SEGClWxp_RbnaQ",
          "id": 24578,
          "type": "transport",
          "action": "cluster:monitor/tasks/lists",
          "start_time_in_millis": 1611612517044,
          "running_time_in_nanos": 638700,
          "cancellable": false,
          "headers": {}
        },
        "Mgqdm0r9SEGClWxp_RbnaQ:24579": {
          "node": "Mgqdm0r9SEGClWxp_RbnaQ",
          "id": 24579,
          "type": "direct",
          "action": "cluster:monitor/tasks/lists[n]",
          "start_time_in_millis": 1611612517044,
          "running_time_in_nanos": 222200,
          "cancellable": false,
          "parent_task_id": "Mgqdm0r9SEGClWxp_RbnaQ:24578",
          "headers": {}
        }
      }
    }
  }
}
```

以下请求返回有关活动搜索任务的详细信息：

#### 示例请求

```bash
curl -XGET "localhost:9200/_tasks?actions=*search&detailed
```
{% include copy.html %}

#### 示例响应

```json
{
  "nodes" : {
    "CRqNwnEeRXOjeTSYYktw-A" : {
      "name" : "runTask-0",
      "transport_address" : "127.0.0.1:9300",
      "host" : "127.0.0.1",
      "ip" : "127.0.0.1:9300",
      "roles" : [
        "cluster_manager",
        "data",
        "ingest",
        "remote_cluster_client"
      ],
      "attributes" : {
        "testattr" : "test",
        "shard_indexing_pressure_enabled" : "true"
      },
      "tasks" : {
        "CRqNwnEeRXOjeTSYYktw-A:677" : {
          "node" : "CRqNwnEeRXOjeTSYYktw-A",
          "id" : 677,
          "type" : "transport",
          "action" : "indices:data/read/search",
          "description" : "indices[], search_type[QUERY_THEN_FETCH], source[{\"query\":{\"query_string\":<QUERY_STRING>}}]",
          "start_time_in_millis" : 1660106254525,
          "running_time_in_nanos" : 1354236,
          "cancellable" : true,
          "cancelled" : false,
          "headers" : { },
          "resource_stats" : {
            "average" : {
              "cpu_time_in_nanos" : 0,
              "memory_in_bytes" : 0
            },
            "total" : {
              "cpu_time_in_nanos" : 0,
              "memory_in_bytes" : 0
            },
            "min" : {
              "cpu_time_in_nanos" : 0,
              "memory_in_bytes" : 0
            },
            "max" : {
              "cpu_time_in_nanos" : 0,
              "memory_in_bytes" : 0
            },
            "thread_info" : {
              "thread_executions" : 0,
              "active_threads" : 0
            }
          }
        }
      }
    }
  }
}

```

### 这`resource_stats` 目的

这`resource_stats` 仅针对支持资源跟踪的任务更新对象。这些统计数据是根据计划的线程执行计算的，包括完成了完成该任务的两个线程和当前从事该任务的线程。由于可以安排同一线程多次处理同一任务，因此安排在给定任务上使用的给定线程的每个实例都被认为是单个线程执行。

下表列出了所有响应字段`resource_stats` 目的。

响应字段| 描述|
:--- | :--- |
`average` | 所有计划的线程执行中的平均资源使用情况。|
`total` | 所有计划的线程执行中的资源使用总和。|
`min` | 所有计划的线程执行中的最小资源使用率。|
`max` | 所有计划的线程执行中的最大资源使用情况。|
`thread_info` | 线-数数-相关统计。|
`thread_info.active_threads` | 当前从事该任务的线程数。|
`thread_info.thread_executions` | 已计划处理该任务的线程数量。|

## 任务取消

获得任务列表后，您可以通过以下请求取消所有可取消任务：

```
POST _tasks/_cancel
```
{% include copy-curl.html %}

请注意，并非所有任务都是可以取消的。要查看任务是否可以取消，请参阅`cancellable` 对您的回应`tasks` API请求。

您还可以通过包括特定任务ID来取消任务。

```
POST _tasks/<task_id>/_cancel
```
{% include copy-curl.html %}

这`cancel` 操作支持与`tasks` 手术。以下示例显示了如何取消多个节点上的所有可取消任务。

```
POST _tasks/_cancel?nodes=opensearch-node1,opensearch-node2
```
{% include copy-curl.html %}

## 将标题附加到任务

要将请求与任务更好地跟踪，您可以提供一个`X-Opaque-Id:<ID_number>` 标题作为您的HTTPS请求阅读器的一部分`curl` 命令。API将在返回的结果中附加指定的标头。

用法：

```bash
curl -i -H "X-Opaque-Id: 111111" "https://localhost:9200/_tasks" -u 'admin:admin' --insecure
```
{% include copy.html %}

这`_tasks` 操作返回以下结果。

```json
HTTP/1.1 200 OK
X-Opaque-Id: 111111
content-type: application/json; charset=UTF-8
content-length: 768

{
  "nodes": {
    "Mgqdm0r9SEGClWxp_RbnaQ": {
      "name": "opensearch-node1",
      "transport_address": "172.18.0.4:9300",
      "host": "172.18.0.4",
      "ip": "172.18.0.4:9300",
      "roles": [
        "data",
        "ingest",
        "master",
        "remote_cluster_client"
      ],
      "tasks": {
        "Mgqdm0r9SEGClWxp_RbnaQ:30072": {
          "node": "Mgqdm0r9SEGClWxp_RbnaQ",
          "id": 30072,
          "type": "direct",
          "action": "cluster:monitor/tasks/lists[n]",
          "start_time_in_millis": 1613166701725,
          "running_time_in_nanos": 245400,
          "cancellable": false,
          "parent_task_id": "Mgqdm0r9SEGClWxp_RbnaQ:30071",
          "headers": {
            "X-Opaque-Id": "111111"
          }
        },
        "Mgqdm0r9SEGClWxp_RbnaQ:30071": {
          "node": "Mgqdm0r9SEGClWxp_RbnaQ",
          "id": 30071,
          "type": "transport",
          "action": "cluster:monitor/tasks/lists",
          "start_time_in_millis": 1613166701725,
          "running_time_in_nanos": 658200,
          "cancellable": false,
          "headers": {
            "X-Opaque-Id": "111111"
          }
        }
      }
    }
  }
}
```
此操作支持与`tasks` 手术。以下示例显示了如何关联`X-Opaque-Id` 有特定的任务：

```bash
curl -i -H "X-Opaque-Id: 123456" "https://localhost:9200/_tasks?nodes=opensearch-node1" -u 'admin:admin' --insecure
```
{% include copy.html %}


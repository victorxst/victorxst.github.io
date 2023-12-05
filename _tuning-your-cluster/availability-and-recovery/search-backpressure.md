---
layout: default
title: 搜索背压
nav_order: 60
has_children: false
parent: 可用性和恢复
redirect_from: 
  - /opensearch/search-backpressure/
---

# 搜索背压

搜索背压是一种用于识别资源的机制-密集的搜索请求并在节点处于胁迫下时取消它们。如果在节点或碎片上的搜索请求违反了资源限制，并且未在一定阈值内恢复，则将被拒绝。这些阈值是动态的，可通过[集群设置](#search-backpressure-settings)。

## 测量资源消耗

为了决定是否应用搜索背压，OpenSearch定期衡量每个搜索请求的以下资源消耗统计信息：

- CPU使用率
- 用法
- 经过时间

观察者线程定期测量节点的资源使用率。如果OpenSearch确定节点处于胁迫下，OpenSearch会检查每个搜索任务和搜索shard任务的资源使用情况，并将其与可配置的阈值进行比较。OpenSearch考虑了CPU使用情况，使用量和经过的时间，并分配每个任务一个取消分数，然后用于取消最多的资源-密集任务。

OpenSearch将取消的数量限制为成功完成任务完成的一部分。此外，它限制了单位时间的取消数量。OpenSearch继续监视和取消任务，直到该节点不再被胁迫。

## 取消查询

如果取消查询，如果某些碎片失败，OpenSearch可能会返回部分结果。如果所有碎片都失败了，OpenSearch将返回从服务器中的错误，类似于以下错误：

```json
{
  "error": {
    "root_cause": [
      {
          "type": "task_cancelled_exception",
          "reason": "cancelled task with reason: cpu usage exceeded [17.9ms >= 15ms], elapsed time exceeded [1.1s >= 300ms]"
      },
      {
          "type": "task_cancelled_exception",
          "reason": "cancelled task with reason: elapsed time exceeded [1.1s >= 300ms]"
      }
    ],
    "type": "search_phase_execution_exception",
    "reason": "all shards failed",
    "phase": "query",
    "grouped": true,
    "failed_shards": [
      {
        "shard": 0,
        "index": "foobar",
        "node": "7yIqOeMfRyWW1rHs2S4byw",
        "reason": {
            "type": "task_cancelled_exception",
            "reason": "cancelled task with reason: cpu usage exceeded [17.9ms >= 15ms], elapsed time exceeded [1.1s >= 300ms]"
        }
      },
      {
        "shard": 1,
        "index": "foobar",
        "node": "7yIqOeMfRyWW1rHs2S4byw",
        "reason": {
            "type": "task_cancelled_exception",
            "reason": "cancelled task with reason: elapsed time exceeded [1.1s >= 300ms]"
        }
      }
    ]
  },
  "status": 500
}
```

## 搜索背压模式

搜索背压运行`monitor_only` （默认），`enforced`， 或者`disabled` 模式。在里面`enforced` 模式，服务器拒绝搜索请求。在里面`monitor_only` 模式，服务器实际上并未取消搜索请求，而是跟踪有关它们的统计信息。您可以在[`search_backpressure.mode`](#search-backpressure-settings) 范围。

## 搜索背压设置

搜索背压将几个设置添加到标准OpenSearch集群设置中。这些设置是动态的，因此您可以在不重新启动群集的情况下更改此功能的默认行为。

环境| 默认| 描述
：--- | ：--- | ：---
search_backpressure.mode| `monitor_only` | 搜索背压[模式](#search-backpressure-modes)。有效值是`monitor_only`，，，，`enforced`， 或者`disabled`。
search_backpressure.cancellation_ratio <br> *在2.6中弃用。替换为search_backpressure.search_shard_task.cancellation_ratio*| 10％| 取消任务的最大任务数量是成功完成任务的百分比。
search_backpressure.cancellation_rate <br> *在2.6中弃用。替换为search_backpressure.search_shard_task.cancellation_rate*| 0.003| 每毫秒段的时间消除的任务数量最大数量。
search_backpressure.cancellation_burst <br> *在2.6中弃用。替换为search_backpressure.search_shard_task.cancellation_burst*| 10| 在观察者线程的单个迭代中取消的最大搜索碎片任务数量。
search_backpressure.node_duress.num_successive_breaches| 3| 连续的极限漏洞的数量之后，该节点被认为处于胁迫之下。
search_backpressure.node_duress.cpu_threshold| 90％| CPU使用阈值（作为百分比）才能被认为是胁迫的。
search_backpressure.node_duress.heap_threshold| 70％| 将节点被视为胁迫所需的堆使用阈值（作为百分比）。
search_backpressure.search_task.elapsed_time_millis_threshold| 45,000| 单个父任务被考虑取消之前所需的经过的时间阈值（以毫秒为单位）。
search_backpressure.search_task.cancellation_ratio| 0.1| 取消搜索任务的最大数量是成功搜索任务完成的百分比。
search_backpressure.search_task.cancellation_rate| 0.003| 每毫秒的经过时间取消的最大搜索任务数量。
search_backpressure.search_task.cancellation_burst| 5| 在观察者线程的单个迭代中取消的最大搜索任务数量。
search_backpressure.search_task.heap_percent_threshold| 2％| 在考虑取消之前，单个父任务所需的堆使用阈值（作为一个百分比）。
search_backpressure.search_task.total_heap_percent_threshold| 5％| 在应用取消任务之前，所有搜索任务的总和之和所需的堆使用阈值（作为百分比）。
search_backpressure.search_task.heap_variance| 2.0| 单个父任务需要在考虑取消之前所需的堆用法差异。考虑到取消任务时`taskHeapUsage` 大于或等于`heapUsageMovingAverage` *`variance`。
search_backpressure.search_task.heap_moving_average_window_size| 10| 用于计算完整父任务的堆滚动使用量的窗口大小。
search_backpressure.search_task.cpu_time_millis_threshold| 30,000| 单个父任务被考虑取消之前所需的CPU使用阈值（以毫秒为单位）。
search_backpressure.search_shard_task.elapsed_time_millis_threshold| 30,000| 单个搜索碎片任务所需的经过的时间阈值（以毫秒为单位）在考虑取消之前。
search_backpressure.search_shard_task.cancellation_ratio| 0.1| 取消的最大搜索碎片任务数量是成功的搜索碎片任务完成的百分比。
search_backpressure.search_shard_task.cancellation_rate| 0.003| 每毫秒的经过时间取消搜索碎片任务数量的最大数量。
search_backpressure.search_shard_task.cancellation_burst| 10| 在观察者线程的单个迭代中取消的最大搜索碎片任务数量。
search_backpressure.search_shard_task.heap_percent_threshold| 0.5％| 单个搜索碎片任务所需的堆使用阈值（作为百分比）在考虑取消之前。
search_backpressure.search_shard_task.total_heap_percent_threshold| 5％| 在应用取消之前，所有搜索碎片任务的堆量总和所需的堆使用阈值（作为百分比）。
search_backpressure.search_shard_task.heap_variance| 2.0| 与先前完成的任务的滚动平均值相比，单个搜索碎片任务的堆积使用量所需的最小差异。
search_backpressure.search_shard_task.heap_moving_average_window_size| 100| 在计算堆用法的滚动平均值时，要考虑的先前完成的搜索碎片任务数量。
search_backpressure.search_shard_task.cpu_time_millis_threshold| 15,000| 单个搜索碎片任务所需的CPU使用阈值（以毫秒为单位）在考虑取消之前。

## 搜索背压统计数据API
引入2.4
{：.label .label-紫色的 }

您可以使用[节点统计API操作]({{site.url}}{{site.baseurl}}/api-reference/nodes-apis/nodes-stats) 监视服务器-侧请求取消。

#### 示例请求

要检索统计信息，请使用以下请求：

```json
GET _nodes/stats/search_backpressure
```

#### 示例响应

响应包含服务器-侧请求取消统计：

```json
{
  "_nodes": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "cluster_name": "runTask",
  "nodes": {
    "T7aqO6zaQX-lt8XBWBYLsA": {
      "timestamp": 1667409521070,
      "name": "runTask-0",
      "transport_address": "127.0.0.1:9300",
      "host": "127.0.0.1",
      "ip": "127.0.0.1:9300",
      "roles": [
         
      ],
      "attributes": {
        "testattr": "test",
        "shard_indexing_pressure_enabled": "true"
      },
      "search_backpressure": {
        "search_task": {
          "resource_tracker_stats": {
            "heap_usage_tracker": {
              "cancellation_count": 57,
              "current_max_bytes": 5739204,
              "current_avg_bytes": 962465,
              "rolling_avg_bytes": 4009239
            },
            "elapsed_time_tracker": {
              "cancellation_count": 97,
              "current_max_millis": 15902,
              "current_avg_millis": 9705
            },
            "cpu_usage_tracker": {
              "cancellation_count": 64,
              "current_max_millis": 8483,
              "current_avg_millis": 7843
            }
          },
          "cancellation_stats": {
            "cancellation_count": 102,
            "cancellation_limit_reached_count": 25
          }
        },
        "search_shard_task": {
          "resource_tracker_stats": {
            "heap_usage_tracker": {
              "cancellation_count": 34,
              "current_max_bytes": 1203272,
              "current_avg_bytes": 700267,
              "rolling_avg_bytes": 1156270
            },
            "cpu_usage_tracker": {
              "cancellation_count": 318,
              "current_max_millis": 731,
              "current_avg_millis": 303
            },
            "elapsed_time_tracker": {
              "cancellation_count": 310,
              "current_max_millis": 1305,
              "current_avg_millis": 649
            }
          },
          "cancellation_stats": {
            "cancellation_count": 318,
            "cancellation_limit_reached_count": 97
          }
        },
        "mode": "enforced"
      }
    }
  }
}
```

### 响应字段

响应包含以下字段。

字段名称| 数据类型| 描述
：--- | ：--- | ：---
search_backpressure| 目的| 有关搜索背压的统计数据。
search_backpressure.search_task| 目的| 搜索任务的统计信息。
search_backpressure.search_task。[Resource_tracker_stats](#resource_tracker_stats) | 目的| 有关当前搜索任务的统计信息。
search_backpressure.search_task。[cancellation_stats](#cancellation_stats) | 目的| 自节点上次重新启动以来，有关搜索任务的统计信息已取消。
search_backpressure.search_shard_task| 目的| 搜索碎片任务的统计信息。
search_backpressure.search_shard_task。[Resource_tracker_stats](#resource_tracker_stats) | 目的| 有关当前搜索碎片任务的统计信息。
search_backpressure.search_shard_task。[cancellation_stats](#cancellation_stats) | 目的|  自节点上次重新启动以来，有关搜索碎片任务的统计信息已取消。
search_backpressure.mode| 细绳| 这[模式](#search-backpressure-modes) 用于搜索背压。

### `resource_tracker_stats`

这`resource_tracker_stats` 对象包含每个资源跟踪器的统计信息：[`elapsed_time_tracker`](#elapsed_time_tracker)，，，，[`heap_usage_tracker`](#heap_usage_tracker)， 和[`cpu_usage_tracker`](#cpu_usage_tracker)。

#### `elapsed_time_tracker`

这`elapsed_time_tracker` 对象包含以下与经过时间有关的统计信息。

字段名称| 数据类型| 描述
：--- | ：--- | ：---
cancellation_count| 整数| 自从节点上次重新启动以来，由于过度经过的时间标记了取消的任务数量。
current_max_millis| 整数| 对于当前在节点上运行的所有任务（以毫秒为单位）的最大时间。
current_avg_millis| 整数| 当前在节点上运行的所有任务的平均时间（以毫秒为单位）。

#### `heap_usage_tracker`

这`heap_usage_tracker` 对象包含与堆用法相关的以下统计信息。

字段名称| 数据类型| 描述
：--- | ：--- | ：---
cancellation_count| 整数| 自从节点上次重新启动以来，由于过度使用堆的过多使用而标记的任务数量。
current_max_bytes| 整数| 当前在节点上运行的所有任务，字节中的所有任务的最大用法。
current_avg_bytes| 整数| 当前在节点上运行的所有任务的平均堆用法，字节。
rolling_avg_bytes| 整数| 滚动的平均堆量`n` 最新任务，字节。`n` 可配置和由`search_backpressure.search_shard_task.heap_moving_average_window_size` 环境。此设置的默认值为100。

#### `cpu_usage_tracker`

这`cpu_usage_tracker` 对象包含以下与CPU使用相关的统计信息。

字段名称| 数据类型| 描述
：--- | ：--- | ：---
cancellation_count| 整数| 自从节点上次重新启动以来的CPU使用过多，因此标记为取消的任务数量。
current_max_millis| 整数| 当前在节点上运行的所有任务的最大CPU时间，以毫秒为单位。
current_avg_millis| 整数| 当前在节点上运行的所有任务的平均CPU时间为毫秒。

### `cancellation_stats`

这`cancellation_stats` 对象包含以下统计信息，用于标记取消的任务。

字段名称| 数据类型| 描述
：--- | ：--- | ：---
cancellation_count| 整数| 自节点上次重新启动以来，标记为取消的任务总数。
CANCELLATION_LIMIT_REACHED_COUNT| 整数| 有资格取消的任务数量超过集合取消阈值的次数


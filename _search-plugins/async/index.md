---
layout: default
title: 异步搜索
nav_order: 51
has_children: true
redirect_from:
  - /search-plugins/async/
---

# 异步搜索

搜索大量数据可能需要很长时间，尤其是当您在温暖的节点或多个远程群集上搜索时。

OpenSearch中的异步搜索使您可以发送在后台运行的搜索请求。您可以监视这些搜索的进度，并在可用时恢复部分结果。搜索完成后，您可以保存结果以稍后检查。

## REST API
引入1.0
{: .label .label-purple }

要执行异步搜索，请将请求发送到`_plugins/_asynchronous_search`，在请求正文中查询您的查询：

```json
POST _plugins/_asynchronous_search
```

您可以指定以下选项。

选项| 描述| 默认值| 必需的
:--- | :--- |:--- |:--- |
`wait_for_completion_timeout` |  您计划等待结果的时间。您可以像在正常搜索中一样看到这段时间内获得的任何结果。您可以根据ID进行轮询剩余结果。最大值为300秒。| 1秒| 不
`keep_on_completion` |  搜索完成后，是否要将结果保存在群集中。您可以在以后检查存储的结果。| `false` | 不
`keep_alive` |  结果保存在群集中的时间。例如，`2d` 意味着结果存储在群集中48小时。保存的搜索结果将在此期间或取消搜索后删除。请注意，这包括查询执行时间。如果查询这次超越，则该过程会自动取消此查询。| 12小时| 不

#### 示例请求

```json
POST _plugins/_asynchronous_search/?pretty&size=10&wait_for_completion_timeout=1ms&keep_on_completion=true&request_cache=false
{
  "aggs": {
    "city": {
      "terms": {
        "field": "city",
        "size": 10
      }
    }
  }
}
```

#### 示例响应

```json
{
  "*id*": "FklfVlU4eFdIUTh1Q1hyM3ZnT19fUVEUd29KLWZYUUI3TzRpdU5wMjRYOHgAAAAAAAAABg==",
  "state": "RUNNING",
  "start_time_in_millis": 1599833301297,
  "expiration_time_in_millis": 1600265301297,
  "response": {
    "took": 15,
    "timed_out": false,
    "terminated_early": false,
    "num_reduce_phases": 4,
    "_shards": {
      "total": 21,
      "successful": 4,
      "skipped": 0,
      "failed": 0
    },
    "hits": {
      "total": {
        "value": 807,
        "relation": "eq"
      },
      "max_score": null,
      "hits": []
    },
    "aggregations": {
      "city": {
        "doc_count_error_upper_bound": 16,
        "sum_other_doc_count": 403,
        "buckets": [
          {
            "key": "downsville",
            "doc_count": 1
          },
        ....
        ....
        ....
          {
            "key": "blairstown",
            "doc_count": 1
          }
        ]
      }
    }
  }
}
```

#### 响应参数

选项| 描述
:--- | :---
`id` | 异步搜索的ID。使用此ID监视搜索的进度，获得部分结果和/或删除结果。如果异步搜索在超时期内完成，则响应不包括ID，因为结果未存储在群集中。
`state` | 指定搜索是否仍在运行，是否已经完成，以及结果是否持续存在。可能的状态是`RUNNING`，`SUCCEEDED`，`FAILED`，`PERSISTING`，`PERSIST_SUCCEEDED`，`PERSIST_FAILED`，`CLOSED` 和`STORE_RESIDENT`。
`start_time_in_millis` | 毫秒开始的开始时间。
`expiration_time_in_millis` | 到期时间以毫秒为单位。
`took` | 搜索正在运行的总时间。
`response` | 实际搜索响应。
`num_reduce_phases` | 协调节点聚集的次数是由于碎片响应的批次（默认情况下为5）。如果该数字与最后检索结果相比增加，则您可以期望将其他结果包括在搜索响应中。
`total` | 运行搜索的碎片总数。
`successful` | 协调节点成功收到的分片响应数量。
`aggregations` | 到目前为止，碎片已经完成的部分聚合结果。

## 获得部分结果
引入1.0
{: .label .label-purple }

提交异步搜索请求后，您可以在异步搜索响应中看到的ID请求部分响应。

```json
GET _plugins/_asynchronous_search/<ID>?pretty
```

#### 示例响应

```json
{
  "id": "Fk9lQk5aWHJIUUltR2xGWnpVcWtFdVEURUN1SWZYUUJBVkFVMEJCTUlZUUoAAAAAAAAAAg==",
  "state": "STORE_RESIDENT",
  "start_time_in_millis": 1599833907465,
  "expiration_time_in_millis": 1600265907465,
  "response": {
    "took": 83,
    "timed_out": false,
    "_shards": {
      "total": 20,
      "successful": 20,
      "skipped": 0,
      "failed": 0
    },
    "hits": {
      "total": {
        "value": 1000,
        "relation": "eq"
      },
      "max_score": 1,
      "hits": [
        {
          "_index": "bank",
          "_id": "1",
          "_score": 1,
          "_source": {
            "email": "amberduke@abc.com",
            "city": "Brogan",
            "state": "IL"
          }
        },
       {....}
      ]
    },
    "aggregations": {
      "city": {
        "doc_count_error_upper_bound": 0,
        "sum_other_doc_count": 997,
        "buckets": [
          {
            "key": "belvoir",
            "doc_count": 2
          },
          {
            "key": "aberdeen",
            "doc_count": 1
          },
          {
            "key": "abiquiu",
            "doc_count": 1
          }
        ]
      }
    }
  }
}
```

响应成功持续后，您将回到`STORE_RESIDENT` 在响应中陈述。

您可以用`wait_for_completion_timeout` 参数等待您指定的时间收到的结果。

用于异步搜索`keep_on_completion` 作为`true` 和足够长的`keep_alive` 时间，您可以继续轮询ID，直到搜索完成为止。如果您不想定期轮询每个ID，则可以将结果保留在群集中`keep_alive` 参数并在以后再回到它。

## 删除搜索和结果
引入1.0
{: .label .label-purple }

删除异步搜索：

```
DELETE _plugins/_asynchronous_search/<ID>?pretty
```

- 如果搜索仍在运行，请搜索取消它。
- 如果搜索完成，则OpenSearch删除保存的结果。


#### 示例响应

```json
{
  "acknowledged": "true"
}
```

## 监视统计数据
引入1.0
{: .label .label-purple }

您可以使用STATS API操作来监视正在运行，完成和/或持续存在的异步搜索。

```json
GET _plugins/_asynchronous_search/stats
```

#### 示例响应

```json
{
  "_nodes": {
    "total": 8,
    "successful": 8,
    "failed": 0
  },
  "cluster_name": "264071961897:asynchronous-search",
  "nodes": {
    "JKEFl6pdRC-xNkKQauy7Yg": {
      "asynchronous_search_stats": {
        "submitted": 18236,
        "initialized": 112,
        "search_failed": 56,
        "search_completed": 56,
        "rejected": 18124,
        "persist_failed": 0,
        "cancelled": 1,
        "running_current": 399,
        "persisted": 100
      }
    }
  }
}
```

#### 响应参数

选项| 描述
:--- | :---
`submitted` | 提交的异步搜索请求的数量。
`initialized` | 初始化的异步搜索请求的数量。
`rejected` | 被拒绝的异步搜索请求的数量。
`search_completed` | 成功响应完成的异步搜索请求的数量。
`search_failed` | 响应失败的异步搜索请求的数量。
`persisted` | 最终结果在集群中成功持续存在的异步搜索请求的数量。
`persist_failed` | 最终结果未能持续存在的异步搜索请求的数量。
`running_current` | 在给定协调器节点上运行的异步搜索请求的数量。
`cancelled` | 搜索运行时取消的异步搜索请求的数量。


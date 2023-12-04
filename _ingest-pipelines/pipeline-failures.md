---
layout: default
title: 处理管道故障
nav_order: 15
redirect_from:
  - /api-reference/ingest-apis/pipeline-failures/
---

# 处理管道故障
**引入 1.0** {：.label .label-purple }

每个引入管道都由一系列处理器组成，这些处理器按顺序应用于文档。如果处理器发生故障，则整个管道将发生故障。有两个选项可用于处理故障：

- **使整个管道失败：**如果处理器发生故障，则整个管道将失败，并且不会为文档编制索引。
- **使当前处理器出现故障，然后继续使用下一个处理器：**如果要继续处理文档，即使其中一个处理器出现故障，这也很有用。

默认情况下，如果其中一个处理器发生故障，引入管道将停止。如果希望管道在处理器发生故障时继续运行，则可以在创建管道时将该处理器的参数设置为 `ignore_failure` `true`：

```json
PUT _ingest/pipeline/my-pipeline/
{
  "description": "Rename 'provider' field to 'cloud.provider'",
  "processors": [
    {
      "rename": {
        "field": "provider",
        "target_field": "cloud.provider",
        "ignore_failure": true
      }
    }
  ]
}
```
{% include copy-curl.html %}

你可以指定在处理器发生故障后立即运行的 `on_failure` 参数。如果你指定 `on_failure` 了，则即使 `on_failure` 配置为空，OpenSearch 也会运行管道中的其他处理器：

```json
PUT _ingest/pipeline/my-pipeline/
{
  "description": "Add timestamp to the document",
  "processors": [
    {
      "date": {
        "field": "timestamp_field",
        "formats": ["yyyy-MM-dd HH:mm:ss"],
        "target_field": "@timestamp",
        "on_failure": [
          {
            "set": {
              "field": "ingest_error",
              "value": "failed"
            }
          }
        ]
      }
    }
  ]
}
```
{% include copy-curl.html %}

如果处理器发生故障，OpenSearch 会记录故障并继续运行搜索管道中的所有剩余处理器。要检查是否有任何故障，可以使用[引入管道指标]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/pipeline-failures/#ingest-pipeline-metrics).{：提示}

## 引入管道指标

若要查看引入管道指标，请使用[节点统计 API]({{site.url}}{{site.baseurl}}/api-reference/nodes-apis/nodes-stats/)：

```json
GET /_nodes/stats/ingest?filter_path=nodes.*.ingest
```
{% include copy-curl.html %}

响应包含所有引入管道的统计信息，例如：

```json
 {
  "nodes": {
    "iFPgpdjPQ-uzTdyPLwQVnQ": {
      "ingest": {
        "total": {
          "count": 28,
          "time_in_millis": 82,
          "current": 0,
          "failed": 9
        },
        "pipelines": {
          "user-behavior": {
            "count": 5,
            "time_in_millis": 0,
            "current": 0,
            "failed": 0,
            "processors": [
              {
                "append": {
                  "type": "append",
                  "stats": {
                    "count": 5,
                    "time_in_millis": 0,
                    "current": 0,
                    "failed": 0
                  }
                }
              }
            ]
          },
           "remove_ip": {
            "count": 5,
            "time_in_millis": 9,
            "current": 0,
            "failed": 2,
            "processors": [
              {
                "remove": {
                  "type": "remove",
                  "stats": {
                    "count": 5,
                    "time_in_millis": 8,
                    "current": 0,
                    "failed": 2
                  }
                }
              }
            ]
          }
        }
      }
    }
  }
}
```

**引入管道故障疑难解答：**你应该做的第一件事是检查日志，查看是否有任何错误或警告可以帮助你确定失败的原因。OpenSearch 日志包含有关失败的提取管道的信息，包括失败的处理器和失败的原因。{：.tip}

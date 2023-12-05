---
layout: default
title: 轮廓
parent: ML Commons API
nav_order: 40
---

# 轮廓

配置文件API操作返回有关ML任务和模型的运行时信息。配置文件操作可以帮助运行时调试模型问题。

## 返回的请求数量

默认情况下，配置文件API监视最后100个请求。要更改监视请求的数量，请更新以下群集设置：

```json
PUT _cluster/settings
{
  "persistent" : {
    "plugins.ml_commons.monitoring_request_count" : 1000000 
  }
}
```

要清除所有监视请求，请设置`plugins.ml_commons.monitoring_request_count` 到`0`。

## 路径和HTTP方法

```json
GET /_plugins/_ml/profile
GET /_plugins/_ml/profile/models
GET /_plugins/_ml/profile/tasks
```

## 路径参数

范围| 数据类型| 描述
:--- | :--- | :---
`model_id` | 细绳| 返回特定模型的运行时数据。您可以将多个串在一起`model_id`s返回多个模型配置文件。
`tasks`| 细绳| 返回特定任务的运行时数据。您可以将多个串在一起`task_id`s返回多个任务配置文件。

### 请求字段

所有个人资料主体请求字段都是可选的。

场地| 数据类型| 描述
:--- | :--- | :--- 
`node_ids` | 细绳| 从特定节点返回所有任务和配置文件。
`model_ids` | 细绳| 返回特定模型的运行时数据。您可以将多个模型ID串在一起以返回多个模型配置文件。
`task_ids` | 细绳| 返回特定任务的运行时数据。您可以将多个任务ID串在一起以返回多个任务配置文件。
`return_all_tasks` | 布尔| 确定请求是否返回所有任务。设置为`false`，任务配置文件被排除在响应之外。
`return_all_models` | 布尔| 确定配置文件请求是否返回所有模型。设置为`false`，模型配置文件被排除在响应之外。

#### 示例请求：在特定节点上返回所有任务和模型

```json
GET /_plugins/_ml/profile
{
  "node_ids": ["KzONM8c8T4Od-NoUANQNGg"],
  "return_all_tasks": true,
  "return_all_models": true
}
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "nodes" : {
    "qTduw0FJTrmGrqMrxH0dcA" : { # node id
      "models" : {
        "WWQI44MBbzI2oUKAvNUt" : { # model id
          "worker_nodes" : [ # routing table
            "KzONM8c8T4Od-NoUANQNGg"
          ]
        }
      }
    },
    ...
    "KzONM8c8T4Od-NoUANQNGg" : { # node id
      "models" : {
        "WWQI44MBbzI2oUKAvNUt" : { # model id
          "model_state" : "DEPLOYED", # model status
          "predictor" : "org.opensearch.ml.engine.algorithms.text_embedding.TextEmbeddingModel@592814c9",
          "worker_nodes" : [ # routing table
            "KzONM8c8T4Od-NoUANQNGg"
          ],
          "predict_request_stats" : { # predict request stats on this node
            "count" : 2, # total predict requests on this node
            "max" : 89.978681, # max latency in milliseconds
            "min" : 5.402,
            "average" : 47.6903405,
            "p50" : 47.6903405,
            "p90" : 81.5210129,
            "p99" : 89.13291418999998
          }
        }
      }
    },
    ...
  }
}
```

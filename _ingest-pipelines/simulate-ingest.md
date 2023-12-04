---
layout: default
title: 模拟管道
nav_order: 11
redirect_from:
  - /opensearch/rest-api/ingest-apis/simulate-ingest/
  - /api-reference/ingest-apis/simulate-ingest/
---

# 模拟管道
**引入 1.0** {：.label .label-purple }

使用模拟引入管道 API 操作来运行或测试管道。

## Path 和 HTTP 方法

以下要求**模拟创建的最新引入管道**：

```
GET _ingest/pipeline/_simulate
POST _ingest/pipeline/_simulate
```
{% include copy-curl.html %}

以下要求**根据流水线 ID 模拟单个流水线**：

```
GET _ingest/pipeline/<pipeline-id>/_simulate
POST _ingest/pipeline/<pipeline-id>/_simulate
```
{% include copy-curl.html %}

## 请求正文字段

下表列出了用于运行管道的请求正文字段。

字段 | 必需 | 类型 | 描述
:--- | :--- | :--- | :---
 `docs` | 必需 | 阵列 | 用于测试管道的文档。
 `pipeline` | 可选 | 对象 | 要模拟的管道。如果未包含管道标识符，则响应将模拟创建的最新管道。

该 `docs` 字段可以包含下表中列出的子字段。

字段 | 必需 | 类型 | 描述
:--- | :--- | :--- | :---
 `source` | 必需 | 对象 | 文档的 JSON 正文。
 `id` | 可选 | 字符串 | 唯一的文档标识符。标识符不能在索引中的其他位置使用。
 `index` | 可选 | 字符串 | 显示文档转换后数据的索引。

## 查询参数

下表列出了用于运行管道的查询参数。

参数 | 类型 | 描述
:--- | :--- | :---
 `verbose` | 布尔值 | 详细模式。显示已执行管道中每个处理器的数据输出。

#### 示例：在路径中指定管道

```json
POST /_ingest/pipeline/my-pipeline/_simulate
{
  "docs": [
    {
      "_index": "my-index",
      "_id": "1",
      "_source": {
        "grad_year": 2024,
        "graduated": false,
        "name": "John Doe"
      }
    },
    {
      "_index": "my-index",
      "_id": "2",
      "_source": {
        "grad_year": 2025,
        "graduated": false,
        "name": "Jane Doe"
      }
    }
  ]
}
```
{% include copy-curl.html %}

该请求返回以下响应：

```json
{
  "docs": [
    {
      "doc": {
        "_index": "my-index",
        "_id": "1",
        "_source": {
          "name": "JOHN DOE",
          "grad_year": 2023,
          "graduated": true
        },
        "_ingest": {
          "timestamp": "2023-06-20T23:19:54.635306588Z"
        }
      }
    },
    {
      "doc": {
        "_index": "my-index",
        "_id": "2",
        "_source": {
          "name": "JANE DOE",
          "grad_year": 2023,
          "graduated": true
        },
        "_ingest": {
          "timestamp": "2023-06-20T23:19:54.635746046Z"
        }
      }
    }
  ]
}
```

#### 示例：详细模式

当在参数设置为 `true` 的情况下运行 `verbose` 上一个请求时，响应将显示每个文档的转换顺序。例如，对于具有 ID `1` 的文档，响应包含按顺序应用管道中每个处理器的结果：

```json
{
  "docs": [
    {
      "processor_results": [
        {
          "processor_type": "set",
          "status": "success",
          "description": "Sets the graduation year to 2023",
          "doc": {
            "_index": "my-index",
            "_id": "1",
            "_source": {
              "name": "John Doe",
              "grad_year": 2023,
              "graduated": false
            },
            "_ingest": {
              "pipeline": "my-pipeline",
              "timestamp": "2023-06-20T23:23:26.656564631Z"
            }
          }
        },
        {
          "processor_type": "set",
          "status": "success",
          "description": "Sets 'graduated' to true",
          "doc": {
            "_index": "my-index",
            "_id": "1",
            "_source": {
              "name": "John Doe",
              "grad_year": 2023,
              "graduated": true
            },
            "_ingest": {
              "pipeline": "my-pipeline",
              "timestamp": "2023-06-20T23:23:26.656564631Z"
            }
          }
        },
        {
          "processor_type": "uppercase",
          "status": "success",
          "doc": {
            "_index": "my-index",
            "_id": "1",
            "_source": {
              "name": "JOHN DOE",
              "grad_year": 2023,
              "graduated": true
            },
            "_ingest": {
              "pipeline": "my-pipeline",
              "timestamp": "2023-06-20T23:23:26.656564631Z"
            }
          }
        }
      ]
    }
  ]
}
```

#### 示例：在请求正文中指定管道

或者，你可以直接在请求正文中指定管道，而无需先创建管道：

```json
POST /_ingest/pipeline/_simulate
{
  "pipeline" :
  {
    "description": "Splits text on whitespace characters",
    "processors": [
      {
        "csv" : {
          "field" : "name",
          "separator": ",",
          "target_fields": ["last_name", "first_name"],
          "trim": true
        }
      },
      {
      "uppercase": {
        "field": "last_name"
      }
    }
    ]
  },
  "docs": [
    {
      "_index": "second-index",
      "_id": "1",
      "_source": {
        "name": "Doe,John"
      }
    },
    {
      "_index": "second-index",
      "_id": "2",
      "_source": {
        "name": "Doe, Jane"
      }
    }
  ]
}
```
{% include copy-curl.html %}

#### 响应

该请求返回以下响应：

```json
{
  "docs": [
    {
      "doc": {
        "_index": "second-index",
        "_id": "1",
        "_source": {
          "name": "Doe,John",
          "last_name": "DOE",
          "first_name": "John"
        },
        "_ingest": {
          "timestamp": "2023-08-24T19:20:44.816219673Z"
        }
      }
    },
    {
      "doc": {
        "_index": "second-index",
        "_id": "2",
        "_source": {
          "name": "Doe, Jane",
          "last_name": "DOE",
          "first_name": "Jane"
        },
        "_ingest": {
          "timestamp": "2023-08-24T19:20:44.816492381Z"
        }
      }
    }
  ]
}
```

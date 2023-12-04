---
layout: default
title: 获取管道
nav_order: 12
redirect_from:
  - /opensearch/rest-api/ingest-apis/get-ingest/
  - /api-reference/ingest-apis/get-ingest/
---

# 获取管道
**引入 1.0** {：.label .label-purple }

使用获取引入管道 API 操作检索有关管道的所有信息。

## 检索有关所有管道的信息

以下示例请求返回有关所有引入管道的信息：

```json
GET _ingest/pipeline/
```
{% include copy-curl.html %}

## 检索有关特定管道的信息

以下示例请求返回有关特定管道的信息，对于此示例，该管道为 `my-pipeline`：

```json
GET _ingest/pipeline/my-pipeline
```
{% include copy-curl.html %}

响应包含管道信息：

```json
{
  "my-pipeline": {
    "description": "This pipeline processes student data",
    "processors": [
      {
        "set": {
          "description": "Sets the graduation year to 2023",
          "field": "grad_year",
          "value": 2023
        }
      },
      {
        "set": {
          "description": "Sets graduated to true",
          "field": "graduated",
          "value": true
        }
      },
      {
        "uppercase": {
          "field": "name"
        }
      }
    ]
  }
}
```

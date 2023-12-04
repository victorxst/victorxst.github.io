---
layout: default
title: 范围
parent: 桶聚合
grand_parent: 聚合
nav_order: 150
redirect_from:
  - /query-dsl/aggregations/bucket/range/
---

# 范围聚集

这`range` 聚合使您可以定义每个存储桶的范围。

例如，您可以找到1000、2000、2000和3000，3000和4000之间的字节数。
在`range` 参数，您可以将范围定义为数组的对象。

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "number_of_bytes_distribution": {
      "range": {
        "field": "bytes",
        "ranges": [
          {
            "from": 1000,
            "to": 2000
          },
          {
            "from": 2000,
            "to": 3000
          },
          {
            "from": 3000,
            "to": 4000
          }
        ]
      }
    }
  }
}
```
{% include copy-curl.html %}

响应包括`from` 关键值并排除`to` 钥匙值：

#### 示例响应

```json
...
"aggregations" : {
  "number_of_bytes_distribution" : {
    "buckets" : [
      {
        "key" : "1000.0-2000.0",
        "from" : 1000.0,
        "to" : 2000.0,
        "doc_count" : 805
      },
      {
        "key" : "2000.0-3000.0",
        "from" : 2000.0,
        "to" : 3000.0,
        "doc_count" : 1369
      },
      {
        "key" : "3000.0-4000.0",
        "from" : 3000.0,
        "to" : 4000.0,
        "doc_count" : 1422
      }
    ]
  }
 }
}
```

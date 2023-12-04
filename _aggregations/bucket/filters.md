---
layout: default
title: 过滤器
parent: 桶聚合
grand_parent: 聚合
nav_order: 60
redirect_from:
  - /query-dsl/aggregations/bucket/filters/
---

# 过滤器聚合

A`filters` 聚集与`filter` 聚合，除了它使您可以使用多个滤镜聚合。
而`filter` 聚集导致一个单一的水桶，`filters` 聚合返回多个存储桶，每个定义过滤器中的一个。

要为所有与任何不匹配任何过滤器查询的文档创建一个存储桶，请设置`other_bucket` 财产为`true`：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "200_os": {
      "filters": {
        "other_bucket": true,
        "filters": [
          {
            "term": {
              "response.keyword": "200"
            }
          },
          {
            "term": {
              "machine.os.keyword": "osx"
            }
          }
        ]
      },
      "aggs": {
        "avg_amount": {
          "avg": {
            "field": "bytes"
          }
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

#### 示例响应

```json
...
"aggregations" : {
  "200_os" : {
    "buckets" : [
      {
        "doc_count" : 12832,
        "avg_amount" : {
          "value" : 5897.852711970075
        }
      },
      {
        "doc_count" : 2825,
        "avg_amount" : {
          "value" : 5620.347256637168
        }
      },
      {
        "doc_count" : 1017,
        "avg_amount" : {
          "value" : 3247.0963618485744
        }
      }
    ]
  }
 }
}
```

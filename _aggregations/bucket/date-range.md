---
layout: default
title: 日期范围
parent: 桶聚合
grand_parent: 聚合
nav_order: 30
redirect_from:
  - /query-dsl/aggregations/bucket/date-range/
---

# 日期范围聚合

这`date_range` 聚集在概念上与`range` 汇总，除非它使您可以执行日期数学。
例如，您可以从过去10天中获取所有文档。为了使日期更可读，请包括一个格式`format` 范围：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "number_of_bytes": {
      "date_range": {
        "field": "@timestamp",
        "format": "MM-yyyy",
        "ranges": [
          {
            "from": "now-10d/d",
            "to": "now"
          }
        ]
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
  "number_of_bytes" : {
    "buckets" : [
      {
        "key" : "03-2021-03-2021",
        "from" : 1.6145568E12,
        "from_as_string" : "03-2021",
        "to" : 1.615451329043E12,
        "to_as_string" : "03-2021",
        "doc_count" : 0
      }
    ]
  }
 }
}
```

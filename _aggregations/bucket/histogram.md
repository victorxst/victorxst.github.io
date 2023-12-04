---
layout: default
title: 直方图
parent: 桶聚合
grand_parent: 聚合
nav_order: 100
redirect_from:
  - /query-dsl/aggregations/bucket/histogram/
---

# 直方图聚集

这`histogram` 基于指定间隔的聚合存储桶文档。

和`histogram` 汇总，您可以很容易地在给定的文档范围内可视化值的分布。现在，OpenSearch并没有给您一个实际的图表，这就是OpenSearch仪表板的目的。但这将为您提供JSON响应，您可以用来构建自己的图形。

以下示例存储桶`number_of_bytes` 划分10,000间隔：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "number_of_bytes": {
      "histogram": {
        "field": "bytes",
        "interval": 10000
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
        "key" : 0.0,
        "doc_count" : 13372
      },
      {
        "key" : 10000.0,
        "doc_count" : 702
      }
    ]
  }
 }
}
```


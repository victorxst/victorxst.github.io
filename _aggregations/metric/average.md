---
layout: default
title: 平均
parent: 公制聚合
grand_parent: 聚合
nav_order: 10
redirect_from:
  - /query-dsl/aggregations/metric/average/
---

# 平均聚合

这`avg` 公制是一个-值度量聚合返回字段的平均值。

以下示例计算`taxful_total_price` 场地：

```json
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "avg_taxful_total_price": {
      "avg": {
        "field": "taxful_total_price"
      }
    }
  }
}
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "took": 85,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4675,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "sum_taxful_total_price": {
      "value": 75.05542864304813
    }
  }
}
```

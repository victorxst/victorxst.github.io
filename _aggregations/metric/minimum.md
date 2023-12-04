---
layout: default
title: 最低限度
parent: 公制聚合
grand_parent: 聚合
nav_order: 70
redirect_from:
  - /query-dsl/aggregations/metric/minimum/
---

# 最小聚合

这`min` 公制是一个-返回字段最小值的值度量聚合。

以下示例计算`taxful_total_price` 场地：

```json
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "min_taxful_total_price": {
      "min": {
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
  "took": 13,
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
    "min_taxful_total_price": {
      "value": 6.98828125
    }
  }
}
```

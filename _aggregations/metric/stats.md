---
layout: default
title: 统计
parent: 公制聚合
grand_parent: 聚合
nav_order: 110
redirect_from:
  - /query-dsl/aggregations/metric/stats/
---

# 统计汇总

这`stats` 公制是多型-价值指标聚合返回所有基本指标，例如`min`，`max`，`sum`，`avg`， 和`value_count` 在一个聚合查询中。

以下示例返回`taxful_total_price` 场地：

```json
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "stats_taxful_total_price": {
      "stats": {
        "field": "taxful_total_price"
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
  "stats_taxful_total_price" : {
    "count" : 4675,
    "min" : 6.98828125,
    "max" : 2250.0,
    "avg" : 75.05542864304813,
    "sum" : 350884.12890625
  }
}
```

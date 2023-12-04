---
layout: default
title: 百分位数
parent: 公制聚合
grand_parent: 聚合
nav_order: 80
redirect_from:
  - /query-dsl/aggregations/metric/percentile-ranks/
---

# 百分位等级聚合

百分位等级是按指定值分组的阈值或低于阈值的百分位数。例如，如果一个值大于或等于值的80％，则其百分比等级为80。

```json
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "percentile_rank_taxful_total_price": {
      "percentile_ranks": {
        "field": "taxful_total_price",
        "values": [
          10,
          15
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
  "percentile_rank_taxful_total_price" : {
    "values" : {
      "10.0" : 0.055096056411283456,
      "15.0" : 0.0830092961834656
    }
  }
}
```

---
layout: default
title: 百分位数
parent: 公制聚合
grand_parent: 聚合
nav_order: 90
redirect_from:
  - /query-dsl/aggregations/metric/percentile/
---

# 百分位聚合

百分位数是在一定阈值值的数据中的百分比。

这`percentile` 公制是多型-值度量集合使您可以在数据中找到离群值或找出数据的分布。

像`cardinality` 公制，`percentile` 公制也近似。

以下示例计算了相对于`taxful_total_price` 场地：

```json
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "percentile_taxful_total_price": {
      "percentiles": {
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
  "percentile_taxful_total_price" : {
    "values" : {
      "1.0" : 21.984375,
      "5.0" : 27.984375,
      "25.0" : 44.96875,
      "50.0" : 64.22061688311689,
      "75.0" : 93.0,
      "95.0" : 156.0,
      "99.0" : 222.0
    }
  }
}
```


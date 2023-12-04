---
layout: default
title: 价值计数
parent: 公制聚合
grand_parent: 聚合
nav_order: 140
redirect_from:
  - /query-dsl/aggregations/metric/value-count/
---

# 值计数聚合

这`value_count` 公制是一个-值度量聚合计算聚合基于的值的数量。

例如，您可以使用`value_count` 与`avg` 指标查找聚合用来计算平均值的数字。

```json
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
   "aggs": {
    "number_of_values": {
      "value_count": {
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
    "number_of_values" : {
      "value" : 4675
    }
  }
```

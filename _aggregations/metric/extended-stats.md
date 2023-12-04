---
layout: default
title: 扩展统计数据
parent: 公制聚合
grand_parent: 聚合
nav_order: 30
redirect_from:
  - /query-dsl/aggregations/metric/extended-stats/
---

# 扩展统计汇总

这`extended_stats` 聚合是一个扩展版本的[`stats`]({{site.url}}{{site.baseurl}}/query-dsl/aggregations/metric/stats/) 聚合。除了包括基本统计数据外，`extended_stats` 还返回统计数据，例如`sum_of_squares`，`variance`， 和`std_deviation`。
以下示例返回了扩展统计信息`taxful_total_price`：
```json
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "extended_stats_taxful_total_price": {
      "extended_stats": {
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
  "extended_stats_taxful_total_price" : {
    "count" : 4675,
    "min" : 6.98828125,
    "max" : 2250.0,
    "avg" : 75.05542864304813,
    "sum" : 350884.12890625,
    "sum_of_squares" : 3.9367749294174194E7,
    "variance" : 2787.59157113862,
    "variance_population" : 2787.59157113862,
    "variance_sampling" : 2788.187974983536,
    "std_deviation" : 52.79764740155209,
    "std_deviation_population" : 52.79764740155209,
    "std_deviation_sampling" : 52.80329511482722,
    "std_deviation_bounds" : {
      "upper" : 180.6507234461523,
      "lower" : -30.53986616005605,
      "upper_population" : 180.6507234461523,
      "lower_population" : -30.53986616005605,
      "upper_sampling" : 180.66201887270256,
      "lower_sampling" : -30.551161586606312
    }
  }
}
```

这`std_deviation_bounds` 对象提供了数据的视觉差异，其中一个间隔为plus/减去两个标准偏差。
将标准偏差设置为不同的值，例如3，设置`sigma` 到3：

```json
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "extended_stats_taxful_total_price": {
      "extended_stats": {
        "field": "taxful_total_price",
        "sigma": 3
      }
    }
  }
}
```

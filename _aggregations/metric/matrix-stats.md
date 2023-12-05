---
layout: default
title: 矩阵统计
parent: 公制聚合
grand_parent: 聚合
nav_order: 50
redirect_from:
  - /query-dsl/aggregations/metric/matrix-stats/
---

# 矩阵统计汇总

这`matrix_stats` 聚合以矩阵形式生成了多个字段的高级统计信息。
以下示例返回以矩阵形式的高级统计`taxful_total_price` 和`products.base_price` 字段：

```json
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "matrix_stats_taxful_total_price": {
      "matrix_stats": {
        "fields": ["taxful_total_price", "products.base_price"]
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
  "matrix_stats_taxful_total_price" : {
    "doc_count" : 4675,
    "fields" : [
      {
        "name" : "products.base_price",
        "count" : 4675,
        "mean" : 34.994239430147196,
        "variance" : 360.5035285833703,
        "skewness" : 5.530161335032702,
        "kurtosis" : 131.16306324042148,
        "covariance" : {
          "products.base_price" : 360.5035285833703,
          "taxful_total_price" : 846.6489362233166
        },
        "correlation" : {
          "products.base_price" : 1.0,
          "taxful_total_price" : 0.8444765264325268
        }
      },
      {
        "name" : "taxful_total_price",
        "count" : 4675,
        "mean" : 75.05542864304839,
        "variance" : 2788.1879749835402,
        "skewness" : 15.812149139924037,
        "kurtosis" : 619.1235507385902,
        "covariance" : {
          "products.base_price" : 846.6489362233166,
          "taxful_total_price" : 2788.1879749835402
        },
        "correlation" : {
          "products.base_price" : 0.8444765264325268,
          "taxful_total_price" : 1.0
        }
      }
    ]
  }
}
```

下表列出了所有响应字段。

统计| 描述
:--- | :---
`count` | 测量样品的数量。
`mean` | 从样品中测量的场的平均值。
`variance` | 测得的场值从其平均值中传播多远。差异越大，它从平均值中传播的越多。
`skewness` | 围绕平均值的场值分布的不对称度量。
`kurtosis` | 分布的尾巴重度的度量。随着尾巴变得更轻，峰度下降。随着尾巴变得更重，峰度增加。要了解峰度，请参阅[维基百科](https://en.wikipedia.org/wiki/Kurtosis)。
`covariance` | 两个字段之间关节变异性的度量。正值意味着它们的价值朝着相同的方向移动，另一方面则移动。
`correlation` | 衡量两个领域之间关系强度的度量。有效值在[-1，1]。一个值-1表示该值是负相关的，而值为1表示其正相关。值为0意味着它们之间没有可识别的关系


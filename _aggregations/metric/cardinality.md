---
layout: default
title: 基数
parent: 公制聚合
grand_parent: 聚合
nav_order: 20
redirect_from:
  - /query-dsl/aggregations/metric/cardinality/
---

# 基数聚合

这`cardinality` 公制是一个-值度量集合计算字段的唯一值或不同值的数量。

以下示例在电子商务商店中找到了独特产品的数量：

```json
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "unique_products": {
      "cardinality": {
        "field": "products.product_id"
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
    "unique_products" : {
      "value" : 7033
    }
  }
```

基数计数是近似值。
如果您的假设商店中有成千上万的产品，则准确的基数计算需要将所有值加载到哈希集中并返回其尺寸。这种方法的扩展不是很好。它需要大量的内存，并可能导致高潜伏期。

您可以控制交易-在记忆和准确性之间关闭`precision_threshold` 环境。此设置定义了阈值，以下计数预计将接近准确。高于此值，计数可能会变得不太准确。默认值的默认值`precision_threshold` 是3,000。最大支持值为40,000。

```json
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "unique_products": {
      "cardinality": {
        "field": "products.product_id",
        "precision_threshold": 10000
      }
    }
  }
}
```

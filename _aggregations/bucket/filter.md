---
layout: default
title: 筛选
parent: 桶聚合
grand_parent: 聚合
nav_order: 50
redirect_from:
  - /query-dsl/aggregations/bucket/filter/
---

# 过滤器聚合

A`filter` 聚合是一个查询子句，就像搜索查询一样 - `match` 或者`term` 或者`range`。您可以使用`filter` 在创建存储桶之前，将整个文档缩小到特定集的汇总。

以下示例显示`avg` 在过滤器的上下文中运行的聚合。这`avg` 聚合仅汇总匹配的文档`range` 询问：

```json
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "low_value": {
      "filter": {
        "range": {
          "taxful_total_price": {
            "lte": 50
          }
        }
      },
      "aggs": {
        "avg_amount": {
          "avg": {
            "field": "taxful_total_price"
          }
        }
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
  "low_value" : {
    "doc_count" : 1633,
    "avg_amount" : {
      "value" : 38.363175998928355
    }
  }
 }
}
```

---
layout: default
title: Global
parent: 桶聚合
grand_parent: 聚合
nav_order: 90
redirect_from:
  - /query-dsl/aggregations/bucket/global/
---

# 全球聚合

这`global` 聚合使您可以摆脱过滤器聚合的聚合上下文。即使您包含了一个缩小一组文档的过滤器查询，`global` 在所有文档上的聚合聚合，好像过滤器查询不存在。它忽略了`filter` 聚集并隐含地假设`match_all` 询问。

以下示例返回`avg` 值的值`taxful_total_price` 索引中所有文档的字段：

```json
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "query": {
    "range": {
      "taxful_total_price": {
        "lte": 50
      }
    }
  },
  "aggs": {
    "total_avg_amount": {
      "global": {},
      "aggs": {
        "avg_price": {
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
  "total_avg_amount" : {
    "doc_count" : 4675,
    "avg_price" : {
      "value" : 75.05542864304813
    }
  }
 }
}
```

您可以看到`taxful_total_price` 场是75.05，而不是38.36`filter` 示例查询匹配时


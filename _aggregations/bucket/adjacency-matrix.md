---
layout: default
title: 邻接矩阵
parent: 桶聚合
grand_parent: 聚合
nav_order: 10
redirect_from:
  - /query-dsl/aggregations/bucket/adjacency-matrix/
---

# 邻接矩阵聚集

这`adjacency_matrix` 聚合使您可以定义过滤器表达式并返回相交过滤器的矩阵-矩阵中的空单元代表一个水桶。您可以找到有多少个文档属于任何过滤器的组合。

使用`adjacency_matrix` 通过将数据视为图表来发现概念如何相关的聚合。

例如，在样本电子商务数据集中，分析了不同制造公司如何相关的：

```json
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "interactions": {
      "adjacency_matrix": {
        "filters": {
          "grpA": {
            "match": {
              "manufacturer.keyword": "Low Tide Media"
            }
          },
          "grpB": {
            "match": {
              "manufacturer.keyword": "Elitelligence"
            }
          },
          "grpC": {
            "match": {
              "manufacturer.keyword": "Oceanavigations"
            }
          }
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

#### 示例响应

 ```JSON
 {
   ...
   "aggregations" ：{
     "interactions" ：{
       "buckets" ：[[
         {
           "key" ："grpA"，
           "doc_count" ：1553
         }，，
         {
           "key" ："grpA&grpB"，
           "doc_count" ：590
         }，，
         {
           "key" ："grpA&grpC"，
           "doc_count" ：329
         }，，
         {
           "key" ："grpB"，
           "doc_count" ：1370
         }，，
         {
           "key" ："grpB&grpC"，
           "doc_count" ：299
         }，，
         {
           "key" ："grpC"，
           "doc_count" ：1218
         }
       这是给出的
     }
   }
 }
```

 Let’s take a closer look at the result:

 ```json
 {
    "key" : "grpA&grpB",
    "doc_count" : 590
 }
 ```

- `grpA`: Products manufactured by Low Tide Media.
- `grpB`: Products manufactured by Elitelligence.
- `590`: Number of products that are manufactured by both.

You can use OpenSearch Dashboards to represent this data with a network graph.

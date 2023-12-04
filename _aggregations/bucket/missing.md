---
layout: default
title: Missing
parent: 桶聚合
grand_parent: 聚合
nav_order: 120
redirect_from:
  - /query-dsl/aggregations/bucket/missing/
---

# 缺少聚合

如果您的索引中有根本不包含汇总字段的文档，或者汇总字段的值为null，请使用`missing` 参数要指定存储桶的名称，应放入此类文档。

以下示例将任何缺少的值添加到名为的存储桶中"N/A"：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "response_codes": {
      "terms": {
        "field": "response.keyword",
        "size": 10,
        "missing": "N/A"
      }
    }
  }
}
```
{% include copy-curl.html %}

因为`min_doc_count` 参数为1，`missing` 参数在其响应中不会返回任何存储桶。放`min_doc_count` 参数到0以查看"N/A" 回应中的水桶：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "response_codes": {
      "terms": {
        "field": "response.keyword",
        "size": 10,
        "missing": "N/A",
        "min_doc_count": 0
      }
    }
  }
}
```

#### 示例响应

```json
...
"aggregations" : {
  "response_codes" : {
    "doc_count_error_upper_bound" : 0,
    "sum_other_doc_count" : 0,
    "buckets" : [
      {
        "key" : "200",
        "doc_count" : 12832
      },
      {
        "key" : "404",
        "doc_count" : 801
      },
      {
        "key" : "503",
        "doc_count" : 441
      },
      {
        "key" : "N/A",
        "doc_count" : 0
      }
    ]
  }
 }
}
```

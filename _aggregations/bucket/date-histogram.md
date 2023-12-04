---
layout: default
title: 日期直方图
parent: 桶聚合
grand_parent: 聚合
nav_order: 20
redirect_from:
  - /query-dsl/aggregations/bucket/date-histogram/
---

# 日期直方图聚合

这`date_histogram` 聚合用途[日期数学]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/date/#date-math) 为时间生成直方图-系列数据。

例如，您可以找到您的网站每月获得多少次命中：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "logs_per_month": {
      "date_histogram": {
        "field": "@timestamp",
        "interval": "month"
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
  "logs_per_month" : {
    "buckets" : [
      {
        "key_as_string" : "2020-10-01T00:00:00.000Z",
        "key" : 1601510400000,
        "doc_count" : 1635
      },
      {
        "key_as_string" : "2020-11-01T00:00:00.000Z",
        "key" : 1604188800000,
        "doc_count" : 6844
      },
      {
        "key_as_string" : "2020-12-01T00:00:00.000Z",
        "key" : 1606780800000,
        "doc_count" : 5595
      }
    ]
  }
}
}
```

响应有三个月的日志。如果绘制这些值，则可以在一个月内看到请求流量到您的网站的峰值和山谷。


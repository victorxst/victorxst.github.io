---
layout: default
title: 多样化的采样器
parent: 桶聚合
grand_parent: 聚合
nav_order: 40
redirect_from:
  - /query-dsl/aggregations/bucket/diversified-sampler/
---

# 多样化的采样器聚集

这`diversified_sampler` 聚集使您可以减少样品库分布的偏差。您可以使用`field` 设置以控制任何共同值的任何一个分片上收集的文档数量：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "sample": {
      "diversified_sampler": {
        "shard_size": 1000,
        "field": "response.keyword"
      },
      "aggs": {
        "terms": {
          "terms": {
            "field": "agent.keyword"
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
  "sample" : {
    "doc_count" : 3,
    "terms" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "Mozilla/5.0 (X11; Linux x86_64; rv:6.0a1) Gecko/20110421 Firefox/6.0a1",
          "doc_count" : 2
        },
        {
          "key" : "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322)",
          "doc_count" : 1
        }
      ]
    }
  }
 }
}
```


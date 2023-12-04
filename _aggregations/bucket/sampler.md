---
layout: default
title: 采样器
parent: 桶聚合
grand_parent: 聚合
nav_order: 170
---

# 采样器聚集

如果您汇总了数百万个文档，则可以使用`sampler` 将其范围降低到一小部分文档的聚合以获得更快的响应。这`sampler` 聚合按顶部选择样品-评分文件。

结果是近似值，但密切表示真实数据的分布。这`sampler` 聚合显着提高了查询性能，但是估计的响应并不完全可靠。

基本语法是：

```json
“aggs”: {
  "SAMPLE": {
    "sampler": {
      "shard_size": 100
    },
    "aggs": {...}
  }
}
```

这`shard_size` 属性告诉Opensearch从每个碎片中收集多少个文档（最多）。

以下示例将每个碎片收集的文档数量限制为1,000，然后用`terms` 聚合：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "sample": {
      "sampler": {
        "shard_size": 1000
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
    "doc_count" : 1000,
    "terms" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "Mozilla/5.0 (X11; Linux x86_64; rv:6.0a1) Gecko/20110421 Firefox/6.0a1",
          "doc_count" : 368
        },
        {
          "key" : "Mozilla/5.0 (X11; Linux i686) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.50 Safari/534.24",
          "doc_count" : 329
        },
        {
          "key" : "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322)",
          "doc_count" : 303
        }
      ]
    }
  }
 }
}
```

---
layout: default
title: 重要术语
parent: 桶聚合
grand_parent: 聚合
nav_order: 180
---

# 重要的术语聚合

这`significant_terms` 聚合使您可以在索引中相对于其余数据中的其余数据中发现异常或有趣的术语出现。

前景集是您过滤的一组文档。背景集是索引中所有文档的集合。
这`significant_terms` 聚合检查了前景集中的所有文档，并与背景集中的文档相比，找到了重大出现的分数。

在示例Web日志数据中，每个文档都有一个字段，其中包含`user-agent` 访客。此示例搜索来自iOS操作系统的所有请求。常规`terms` 该前景集合的聚合返回Firefox，因为它在此存储桶中具有最多的文档。另一方面，`significant_terms` 聚合返回Internet Explorer（IE），因为与背景集相比，IE在前景集中的外观明显更高。

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "query": {
    "terms": {
      "machine.os.keyword": [
        "ios"
      ]
    }
  },
  "aggs": {
    "significant_response_codes": {
      "significant_terms": {
        "field": "agent.keyword"
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
  "significant_response_codes" : {
    "doc_count" : 2737,
    "bg_count" : 14074,
    "buckets" : [
      {
        "key" : "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322)",
        "doc_count" : 818,
        "score" : 0.01462731514608217,
        "bg_count" : 4010
      },
      {
        "key" : "Mozilla/5.0 (X11; Linux x86_64; rv:6.0a1) Gecko/20110421 Firefox/6.0a1",
        "doc_count" : 1067,
        "score" : 0.009062566630410223,
        "bg_count" : 5362
      }
    ]
  }
}
```

如果是`significant_terms` 聚合不会返回任何结果，您可能没有通过查询过滤结果。另外，前景集中的项分布可能与背景集相同，这意味着前景集中没有任何异常。

背景术语频率的统计信息的默认来源是整个索引。您可以使用背景过滤器缩小此范围，以获得更多焦点



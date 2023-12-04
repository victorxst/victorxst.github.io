---
layout: default
title: IP范围
parent: 桶聚合
grand_parent: 聚合
nav_order: 110
redirect_from:
  - /query-dsl/aggregations/bucket/ip-range/
---

# IP范围聚集

这`ip_range` 聚合是用于IP地址的。
它可以使用`ip` 类型字段。您可以在[cidr](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) 符号。

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "access": {
      "ip_range": {
        "field": "ip",
        "ranges": [
          {
            "from": "1.0.0.0",
            "to": "126.158.155.183"
          },
          {
            "mask": "1.0.0.0/8"
          }
        ]
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
  "access" : {
    "buckets" : [
      {
        "key" : "1.0.0.0/8",
        "from" : "1.0.0.0",
        "to" : "2.0.0.0",
        "doc_count" : 98
      },
      {
        "key" : "1.0.0.0-126.158.155.183",
        "from" : "1.0.0.0",
        "to" : "126.158.155.183",
        "doc_count" : 7184
      }
    ]
  }
}
```

如果将带有畸形字段的文档添加到具有`ip_range` 设置`false` 在映射中，OpenSearch拒绝了整个文档。您可以设置`ignore_malformed` 到`true` 要指定OpenSearch应该忽略畸形字段。默认值为`false`。

```json
...
"mappings": {
  "properties": {
    "ips": {
      "type": "ip_range",
      "ignore_malformed": true
    }
  }
}
```

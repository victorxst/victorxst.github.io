---
layout: default
title: Multi-terms
parent: 桶聚合
grand_parent: 聚合
nav_order: 130
redirect_from:
  - /query-dsl/aggregations/multi-terms/
---

# 并发-术语聚合

类似于`terms` 存储桶汇总，您还可以使用`multi_terms` 聚合。并发-当您需要按文档计数进行排序，或者需要按照复合键上的度量聚合并获得顶部时，术语聚合很有用`n` 结果。例如，您可以搜索特定数量的文档（例如1000），以及显示CPU使用情况大于90％的每个位置的服务器数量。最高的结果将返回此Multi-术语查询。

这`multi_terms` 聚集确实比一个`terms` 聚合，因此其性能可能会较慢。
{: .tip }

## 并发-术语聚合参数

范围| 描述
:--- | :---
多_terms| 指示一个并发-根据多个术语指定的标准，将汇总的术语聚集在一起。
尺寸| 指定要返回的存储桶数。默认值为10。
命令| 指示对存储桶进行排序的顺序。默认情况下，根据每个存储桶的文档计数排序存储桶。如果存储桶包含相同的文档计数，则`order` 可以明确设置为术语值而不是文档计数。（例如，设置`order` 到"max-cpu"）。
DOC_COUNT| 指定每个存储桶中要退还的文档数量。默认情况下，返回前10个条款。

#### 示例请求

```json
GET sample-index100/_search
{
  "size": 0, 
  "aggs": {
    "hot": {
      "multi_terms": {
        "terms": [{
          "field": "region" 
        },{
          "field": "host" 
        }],
        "order": {"max-cpu": "desc"}
      },
      "aggs": {
        "max-cpu": { "max": { "field": "cpu" } }
      }      
    }
  }
}
```
{% include copy-curl.html %}

#### 示例响应

```json
{
  "took": 118,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 8,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "multi-terms": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": [
            "dub",
            "h1"
          ],
          "key_as_string": "dub|h1",
          "doc_count": 2,
          "max-cpu": {
            "value": 90.0
          }
        },
        {
          "key": [
            "dub",
            "h2"
          ],
          "key_as_string": "dub|h2",
          "doc_count": 2,
          "max-cpu": {
            "value": 70.0
          }
        },
        {
          "key": [
            "iad",
            "h2"
          ],
          "key_as_string": "iad|h2",
          "doc_count": 2,
          "max-cpu": {
            "value": 50.0
          }
        },
        {
          "key": [
            "iad",
            "h1"
          ],
          "key_as_string": "iad|h1",
          "doc_count": 2,
          "max-cpu": {
            "value": 15.0
          }
        }
      ]
    }
  }
}
```


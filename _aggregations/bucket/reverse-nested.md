---
layout: default
title: 反向嵌套
parent: 桶聚合
grand_parent: 聚合
nav_order: 160
redirect_from:
  - /query-dsl/aggregations/bucket/reverse-nested/
---

# 反向嵌套聚集

您可以从嵌套文档到其父母汇总值；这个聚合称为`reverse_nested`。
您可以使用`reverse_nested` 从嵌套对象通过字段分组后，从父文档中汇总字段。这`reverse_nested` 聚合"joins back" 根页并获取`load_time` 对于您的变化。

这`reverse_nested` 聚合是一个子-嵌套聚合中的聚集。它接受一个名称的单个选项`path`。此选项定义了文档层次结构opensearch中的倒数几个步骤以计算聚合。

```json
GET logs/_search
{
  "query": {
    "match": { "response": "200" }
  },
  "aggs": {
    "pages": {
      "nested": {
        "path": "pages"
      },
      "aggs": {
        "top_pages_per_load_time": {
          "terms": {
            "field": "pages.load_time"
          },
          "aggs": {
            "comment_to_logs": {
              "reverse_nested": {},
              "aggs": {
                "min_load_time": {
                  "min": {
                    "field": "pages.load_time"
                  }
                }
              }
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

```json
...
"aggregations" : {
  "pages" : {
    "doc_count" : 2,
    "top_pages_per_load_time" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 200.0,
          "doc_count" : 1,
          "comment_to_logs" : {
            "doc_count" : 1,
            "min_load_time" : {
              "value" : null
            }
          }
        },
        {
          "key" : 500.0,
          "doc_count" : 1,
          "comment_to_logs" : {
            "doc_count" : 1,
            "min_load_time" : {
              "value" : null
            }
          }
        }
      ]
    }
  }
}
```

响应显示日志索引具有一个页面`load_time` 200，一个`load_time` 500


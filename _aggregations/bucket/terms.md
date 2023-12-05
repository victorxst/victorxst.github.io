---
layout: default
title: 术语
parent: 桶聚合
grand_parent: 聚合
nav_order: 200
---

# 术语聚合

这`terms` 聚合会为字段的每个唯一项动态创建一个存储桶。

以下示例使用`terms` 汇总以在Web日志数据中查找每个响应代码的文档数量：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "response_codes": {
      "terms": {
        "field": "response.keyword",
        "size": 10
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
      }
    ]
  }
 }
}
```

值用密钥返回值`key`。
`doc_count` 指定每个存储桶中的文档数量。默认情况下，存储桶以降序排序的顺序`doc-count`。

响应还包括两个键`doc_count_error_upper_bound` 和`sum_other_doc_count`。

这`terms` 聚合返回最高的唯一术语。因此，如果数据具有许多独特的术语，那么结果可能不会出现。这`sum_other_doc_count` 字段是响应中排除的文档的总和。在这种情况下，数字为0，因为所有唯一值都出现在响应中。

这`doc_count_error_upper_bound` 字段表示最终结果中遗留下来的唯一值的最大计数。使用此字段来估计计数的误差余量。

计数可能不准确。负责聚合的协调节点促使每个碎片的最佳术语。想象一个场景`size` 参数为3。
这`terms` 汇总请求每个碎片的前3个唯一术语。协调节点将获取每个结果并将它们汇总以计算最终结果。如果碎片具有不属于前三名的对象，则不会显示在响应中。

如果这尤其如此`size` 设置为较低的数字。由于默认大小为10，因此不太可能发生错误。如果您不需要高准确性并想提高性能，则可以降低尺寸。

## 账户-汇总数据

而`doc_count` 字段提供了在存储桶中汇总的单个文档数量的表示，`doc_count` 本身没有办法正确地增加存储PRE的文档-汇总数据。考虑预先考虑-汇总数据并准确计算存储桶中的文档数量，您可以使用`_doc_count` 字段以在单个摘要字段中添加文档数量。当文件包括`_doc_count` 字段，所有水桶聚合都识别其价值并增加了桶`doc_count` 累积。使用这些考虑时要牢记这些考虑`_doc_count` 场地：

*该字段不支持嵌套数组；只能使用积极的整数。
*如果文档不包含`_doc_count` 字段，聚合使用该文档将计数增加1。

依赖精确文档计数的OpenSearch功能说明了使用`_doc_count` 场地。要查看该字段如何用于支持其他搜索工具，请参阅[索引汇总](https://opensearch.org/docs/latest/im-plugin/index-rollups/index/)，用于索引管理（IM）插件的OpenSearch功能，该插件将文档存储使用PRE-汇总索引中的汇总数据。
{: .tip}

#### 示例请求

```json
PUT /my_index/_doc/1
{
  "response_code": 404,
  "date":"2022-08-05",
  "_doc_count": 20
}

PUT /my_index/_doc/2
{
  "response_code": 404,
  "date":"2022-08-06",
  "_doc_count": 10
}

PUT /my_index/_doc/3
{
  "response_code": 200,
  "date":"2022-08-06",
  "_doc_count": 300
}

GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "response_codes": {
      "terms": {
        "field" : "response_code"
      }
    }
  }
}
```

#### 示例响应

```json
{
  "took" : 20,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "response_codes" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 200,
          "doc_count" : 300
        },
        {
          "key" : 404,
          "doc_count" : 30
        }
      ]
    }
  }
}
```

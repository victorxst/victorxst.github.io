---
layout: default
title: 重要的文字
parent: 桶聚合
grand_parent: 聚合
nav_order: 190
---

# 重要的文本聚合

这`significant_text` 聚集与`significant_terms` 聚合，但这是针对原始文本字段。
重要的文本衡量了使用统计分析在前景集和背景集之间测量的流行度的变化。例如，当您查找其股票首字母缩写TSLA时，它可能建议Tesla。

这`significant_text` 聚合-即时分析源文本，过滤嘈杂的数据，例如重复的段落，样板标头和页脚等等，等等，否则可能会偏向结果。

关于-分析高-基数数据集可以是非常CPU-密集操作。我们建议使用`significant_text` 采样器聚集中的聚集，以将分析限制为一小部分顶部-匹配文档，例如200。

您可以设置以下参数：

- `min_doc_count` - 返回结果匹配的结果超过了配置的最高命中次数。我们建议不要设置`min_doc_count` 到1，因为它倾向于返回错别字或拼写错误的术语。寻找多个术语的实例有助于加强意义不是一个的结果-事故。默认值3用于提供最小的重量-的-证据。
- `shard_size` - 设置高价值会以计算性能为代价提高稳定性（和准确性）。
- `shard_min_doc_count` - 如果您的文本包含许多低频单词，并且您对这些单词不感兴趣（例如错别字），那么您可以设置`shard_min_doc_count` 参数可以在碎片级别过滤候选术语，并具有合理的确定性，无法达到所需`min_doc_count` 即使在合并了局部重要的文本频率之后。默认值为1，直到您明确设置它，它都不会影响。我们建议设置此值远低于`min_doc_count` 价值。

假设您在OpenSearch集群中拥有莎士比亚索引的完整作品。您可以找到与单词有关的重要文本"breathe" 在里面`text_entry` 场地：

```json
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": "breathe"
    }
  },
  "aggregations": {
    "my_sample": {
      "sampler": {
        "shard_size": 100
      },
      "aggregations": {
        "keywords": {
          "significant_text": {
            "field": "text_entry",
            "min_doc_count": 4
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
"aggregations" : {
  "my_sample" : {
    "doc_count" : 59,
    "keywords" : {
      "doc_count" : 59,
      "bg_count" : 111396,
      "buckets" : [
        {
          "key" : "breathe",
          "doc_count" : 59,
          "score" : 1887.0677966101694,
          "bg_count" : 59
        },
        {
          "key" : "air",
          "doc_count" : 4,
          "score" : 2.641295376716233,
          "bg_count" : 189
        },
        {
          "key" : "dead",
          "doc_count" : 4,
          "score" : 0.9665839666414213,
          "bg_count" : 495
        },
        {
          "key" : "life",
          "doc_count" : 5,
          "score" : 0.9090787433467572,
          "bg_count" : 805
        }
      ]
    }
  }
 }
}
```

与`breathe` 是`air`，`dead`， 和`life`。

这`significant_text` 聚合具有以下局限性：

- 不支持儿童聚集，因为儿童聚集的成本很高。作为解决方法，您可以添加关注-使用一个查询`terms` 包含一个包括子句和儿童汇总的汇总。
- 不支持嵌套对象，因为它可以与文档JSON源一起使用。
- 文件计数可能存在一些（通常很小）的错误，因为它基于每个碎片返回的样本。您可以使用`shard_size` 参数到罚款-调整贸易-在准确性和性能之间关闭。默认情况下，`shard_size` 被设定为-1自动估计碎片的数量和`size` 范围。

背景术语频率的统计信息的默认来源是整个索引。您可以使用背景过滤器范围缩小此范围，以获得更多的重点：

```json
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": "breathe"
    }
  },
  "aggregations": {
    "my_sample": {
      "sampler": {
        "shard_size": 100
      },
      "aggregations": {
        "keywords": {
          "significant_text": {
            "field": "text_entry",
            "background_filter": {
              "term": {
                "speaker": "JOHN OF GAUNT"
              }
            }
          }
        }
      }
    }
  }
}
```

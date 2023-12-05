---
layout: default
title: 正常化
nav_order: 15
has_children: false
parent: 搜索处理器
grand_parent: 搜索管道
---

# 归一化处理器

这`normalization-processor` 是搜索阶段结果处理器，在搜索执行的查询阶段和获取阶段之间运行。它拦截了查询阶段结果，然后将文档分数从不同的查询子句中进行归一化，然后再将文档传递给获取阶段。

## 得分归一化和组合

许多应用程序都需要关键字匹配和语义理解。例如，BM25准确地为包含关键字的查询提供了相关的搜索结果，并且当查询需要自然语言了解时，神经网络的表现良好。因此，您可能需要将BM25搜索结果与k的结果相结合-NN或神经搜索。但是，BM25和K-NN搜索使用不同的量表来计算匹配文档的相关性分数。在结合来自多个查询的分数之前，将它们归一化是有益的，以使它们处于相同的尺度上，如实验数据所示。要进一步阅读有关得分归一化和组合的进一步阅读，包括基准和各种技术，请参见[这篇语义搜索博客文章](https://opensearch.org/blog/semantic-science-benchmarks/)。

## 查询然后获取

OpenSearch支持两种搜索类型：`query_then_fetch` 和`dfs_query_then_fetch`。下图概述了查询-然后-提取过程，其中包括标准化处理器。

![标准化处理器流程图]({{site.url}}{{site.baseurl}}/images/normalization-processor.png)

当您将搜索请求发送到节点时，节点将变为_coordinating node_。在搜索的第一个阶段，_Query Phase_，协调节点将搜索请求路由到索引中的所有碎片，包括主和副本碎片。然后，每个碎片在本地运行搜索查询，并返回有关匹配文档的元数据，其中包括其文档ID和相关性分数。这`normalization-processor` 然后将归一化并结合来自不同查询子句的分数。协调节点合并并分类结果的本地列表，并汇总了与查询相匹配的顶级文档的全局列表。之后，搜索执行输入_fetch阶段_，其中协调节点从其居住的碎片中请求全局列表中的文档。每个碎片都返回文档`_source` 到协调节点。最后，协调节点将包含结果的搜索响应发送给您。

## 请求字段

下表列出了所有可用的请求字段。

场地| 数据类型| 描述
:--- | :--- | :---
`normalization.technique` | 细绳| 标准分数的技术。有效值是[`min_max`](https://en.wikipedia.org/wiki/Feature_scaling#Rescaling_(min-max_normalization)） 和[`l2`](https://en.wikipedia.org/wiki/Cosine_similarity#L2-normalized_Euclidean_distance)。选修的。默认为`min_max`。
`combination.technique` | 细绳| 组合分数的技术。有效值是[`arithmetic_mean`](https://en.wikipedia.org/wiki/Arithmetic_mean)，[`geometric_mean`](https://en.wikipedia.org/wiki/Geometric_mean)， 和[`harmonic_mean`](https://en.wikipedia.org/wiki/Harmonic_mean)。选修的。默认为`arithmetic_mean`。
`combination.parameters.weights` | 浮动阵列-点值| 指定每个查询要使用的权重。有效值在[0.0，1.0]范围内，表示十进制百分比。重量越接近1.0，对查询的重量就越多。在`weights` 数组必须等于查询数。数组中的值的总和必须等于1.0。选修的。如果未提供，则所有查询都具有相等的重量。
`tag` | 细绳| 处理器的标识符。选修的。
`description` | 细绳| 处理器的描述。选修的。
`ignore_failure` | 布尔| 对于此处理器，该值将被忽略。如果处理器失败，管道总是会失败并返回错误。

## 例子

以下示例证明了使用搜索管道与`normalization-processor`。要尝试此示例，请按照[语义搜索教程]({{site.url}}{{site.baseurl}}/ml-commons-plugin/semantic-search#tutorial)。

### 创建搜索管道

以下请求创建一个包含一个的搜索管道`normalization-processor` 使用`min_max` 标准化技术和`arithmetic_mean` 组合技术：

```json
PUT /_search/pipeline/nlp-search-pipeline
{
  "description": "Post processor for hybrid search",
  "phase_results_processors": [
    {
      "normalization-processor": {
        "normalization": {
          "technique": "min_max"
        },
        "combination": {
          "technique": "arithmetic_mean",
          "parameters": {
            "weights": [
              0.3,
              0.7
            ]
          }
        }
      }
    }
  ]
}
```
{% include copy-curl.html %}

### 使用搜索管道

提供您要合并的查询条款`hybrid` 查询并应用上一节中创建的搜索管道，以便使用所选技术合并得分：

```json
GET /my-nlp-index/_search?search_pipeline=nlp-search-pipeline
{
  "_source": {
    "exclude": [
      "passage_embedding"
    ]
  },
  "query": {
    "hybrid": {
      "queries": [
        {
          "match": {
            "text": {
              "query": "horse"
            }
          }
        },
        {
          "neural": {
            "passage_embedding": {
              "query_text": "wild west",
              "model_id": "aVeif4oB5Vm0Tdw8zYO2",
              "k": 5
            }
          }
        }
      ]
    }
  }
}
```
{% include copy-curl.html %}

有关更多信息，请参阅[混合查询]({{site.url}}{{site.baseurl}}/query-dsl/compound/hybrid/)。

## 搜索调整建议

为了提高搜索相关性，我们建议增加样本量。

如果混合查询未返回一些预期的结果，则可能是因为子征服返回的文档太少。这`normalization-processor` 仅改变每个子查询返回的结果；它没有执行任何其他抽样。在实验中，我们使用了[NDCG@10](https://en.wikipedia.org/wiki/Discounted_cumulative_gain) 根据返回的文档数量（大小）来衡量信息检索的质量。我们发现[100，200]范围中的大小最适合多达1000万个文档的数据集。我们不建议将尺寸增加到推荐值之外，因为更高的尺寸值不会提高搜索相关性，而是会增加搜索延迟。


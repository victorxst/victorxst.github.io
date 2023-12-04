---
layout: default
title: 解释
nav_order: 30
redirect_from: 
 - /opensearch/rest-api/explain/
---

# 解释
**引入1.0**
{: .label .label-purple }

想知道为什么特定文档对查询排名更高（或更低）？您可以使用解释API来解释相关性分数（`_score`）计算每个结果。

OpenSearch使用概率排名框架称为[Okapi BM25](https://en.wikipedia.org/wiki/Okapi_BM25) 计算相关得分。Okapi BM25基于原始[TF/IDF](https://lucene.apache.org/core/{{site.lucene_version}}/core/org/apache/lucene/search/package-summary.html#scoring) Apache Lucene使用的框架。

就资源和时间而言，解释API都是昂贵的操作。在生产簇上，我们建议将其谨慎地用于故障排除。
{: .warning }


## 例子

要查看所有结果的解释输出，请设置`explain` 标记为`true` 在URL或请求正文中：

```json
POST opensearch_dashboards_sample_data_ecommerce/_search?explain=true
{
  "query": {
    "match": {
      "customer_first_name": "Mary"
    }
  }
}
```
{% include copy-curl.html %}

您经常需要一个文档的输出。在这种情况下，在URL中指定文档ID：

```json
POST opensearch_dashboards_sample_data_ecommerce/_explain/EVz1Q3sBgg5eWQP6RSte
{
  "query": {
    "match": {
      "customer_first_name": "Mary"
    }
  }
}
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET <target>/_explain/<id>
POST <target>/_explain/<id>
```

## URL参数

您必须指定索引和文档ID。所有其他URL参数都是可选的。

范围| 类型| 描述| 必需的
:--- | :--- | :--- | :---
`<index>` | 细绳| 索引的名称。您只能指定一个索引。| 是的
`<_id>` | 细绳| 一个唯一的标识符，可以连接到文档上。| 是的
`analyzer` | 细绳| 用于查询字符串中的分析仪。| 不
`analyze_wildcard` | 布尔| 指定是否分析通配符和前缀查询。默认值为false。| 不
`default_operator` | 细绳| 指示字符串查询的默认运算符应该是和或或。默认为或。| 不
`df` | 细绳| 如果查询字符串中未提供字段前缀，则默认字段。| 不
`lenient` | 布尔| 指定OpenSearch是否应忽略格式-基于查询失败（例如，查询整数的文本字段）。默认值为false。| 不
`preference` | 细绳| 指定偏爱哪些碎片以检索结果。可用选项是`_local`，该操作告诉操作以从本地分配的碎片副本和分配给特定碎片副本的自定义字符串值来检索结果。默认情况下，OpenSearch在随机碎片上执行解释操作。| 不
`q` | 细绳| 在Lucene查询字符串语法中查询。| 不
`stored_fields` | 布尔| 如果是真的，该操作将检索存储在索引中而不是文档的文档字段`_source`。默认值为false。| 不
`routing` | 细绳| 用于将操作路由到特定碎片的值。| 不
`_source` | 细绳| 是否包括`_source` 响应体中的田地。默认是正确的。| 不
`_source_excludes` | 细绳| 逗号-分开的源字段列表要在查询响应中排除。| 不
`_source_includes` | 细绳| 逗号-分开的源字段列表要包括在查询响应中。| 不

## 回复

```json
{
  "_index" : "kibana_sample_data_ecommerce",
  "_id" : "EVz1Q3sBgg5eWQP6RSte",
  "matched" : true,
  "explanation" : {
    "value" : 3.5671005,
    "description" : "weight(customer_first_name:mary in 1) [PerFieldSimilarity], result of:",
    "details" : [
      {
        "value" : 3.5671005,
        "description" : "score(freq=1.0), computed as boost * idf * tf from:",
        "details" : [
          {
            "value" : 2.2,
            "description" : "boost",
            "details" : [ ]
          },
          {
            "value" : 3.4100041,
            "description" : "idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:",
            "details" : [
              {
                "value" : 154,
                "description" : "n, number of documents containing term",
                "details" : [ ]
              },
              {
                "value" : 4675,
                "description" : "N, total number of documents with field",
                "details" : [ ]
              }
            ]
          },
          {
            "value" : 0.47548598,
            "description" : "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
            "details" : [
              {
                "value" : 1.0,
                "description" : "freq, occurrences of term within document",
                "details" : [ ]
              },
              {
                "value" : 1.2,
                "description" : "k1, term saturation parameter",
                "details" : [ ]
              },
              {
                "value" : 0.75,
                "description" : "b, length normalization parameter",
                "details" : [ ]
              },
              {
                "value" : 1.0,
                "description" : "dl, length of field",
                "details" : [ ]
              },
              {
                "value" : 1.1206417,
                "description" : "avgdl, average length of field",
                "details" : [ ]
              }
            ]
          }
        ]
      }
    ]
  }
}
```

## 响应身体场

场地| 描述
:--- | :---
`matched` | 指示该文档是否与查询匹配。
`explanation` | 这`explanation` 对象具有三个属性：`value`，`description`， 和`details`。这`value` 显示了计算的结果`description` 说明执行了哪种类型的计算，然后`details` 显示进行的任何亚估计。
术语频率（`tf`）| 该术语出现在给定文档的字段中多少次。术语发生的次数越多，相关得分就越高。
逆文档频率（`idf`）| 该术语多久出现在索引中（在所有文档中）。术语出现的频率越低是相关得分。
场归一化因子（`fieldNorm`）| 场的长度。OpenSearch分配了更高的相关性分数，该术语出现在相对较短的字段中。

这`tf`，`idf`， 和`fieldNorm` 值计算并存储在添加或更新文档时的索引时间。这些值可能存在一些（通常很小）的不准确性，因为它是基于从每个碎片中返回的样本的总结。

各个查询包括用于计算相关性评分的其他因素，例如术语接近性，模糊性等。


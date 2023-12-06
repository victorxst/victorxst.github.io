---
layout: default
title: 轮廓
nav_order: 55
---

# 轮廓
**引入1.0**
{: .label .label-purple }

配置文件API提供了有关执行搜索请求的各个组件的时序信息。使用配置文件API，您可以调试慢速请求并了解如何提高其性能。配置文件API未衡量以下内容：

- 网络延迟
- 在搜索提取阶段花费的时间
- 请求在队列中花费的时间
- 在协调节点上合并碎片响应时空闲时间

配置文件API是资源-消费操作为搜索操作增加了开销。
{: .warning}

#### 示例请求

要使用配置文件API，请包括`profile` 参数设置为`true` 在发送到的搜索请求中`_search` 端点：

```json
GET /testindex/_search
{
  "profile": true,
  "query" : {
    "match" : { "title" : "wind" }
  }
}
```
{% include copy-curl.html %}

打开人类-可读格式，包括`?human=true` 请求中查询参数：

```json
GET /testindex/_search?human=true
{
  "profile": true,
  "query" : {
    "match" : { "title" : "wind" }
  }
}
```
{% include copy-curl.html %}

响应包含附加`time` 人类的领域-例如，可读单元：

```json
"collector": [
    {
        "name": "SimpleTopScoreDocCollector",
        "reason": "search_top_hits",
        "time": "113.7micros",
        "time_in_nanos": 113711
    }
]
```

配置文件API响应是冗长的，因此，如果您正在通过`curl` 命令，包括`?pretty` 查询参数以使响应易于理解。
{: .tip}

#### 示例响应

响应包含分析信息：

<details closed markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}

```json
{
  "took": 21,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.19363807,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.19363807,
        "_source": {
          "title": "The wind rises"
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.17225474,
        "_source": {
          "title": "Gone with the wind",
          "description": "A 1939 American epic historical film"
        }
      }
    ]
  },
  "profile": {
    "shards": [
      {
        "id": "[LidyZ1HVS-u93-73Z49dQg][testindex][0]",
        "inbound_network_time_in_millis": 0,
        "outbound_network_time_in_millis": 0,
        "searches": [
          {
            "query": [
              {
                "type": "BooleanQuery",
                "description": "title:wind title:rise",
                "time_in_nanos": 2473919,
                "breakdown": {
                  "set_min_competitive_score_count": 0,
                  "match_count": 0,
                  "shallow_advance_count": 0,
                  "set_min_competitive_score": 0,
                  "next_doc": 5209,
                  "match": 0,
                  "next_doc_count": 2,
                  "score_count": 2,
                  "compute_max_score_count": 0,
                  "compute_max_score": 0,
                  "advance": 9209,
                  "advance_count": 2,
                  "score": 20751,
                  "build_scorer_count": 4,
                  "create_weight": 1404458,
                  "shallow_advance": 0,
                  "create_weight_count": 1,
                  "build_scorer": 1034292
                },
                "children": [
                  {
                    "type": "TermQuery",
                    "description": "title:wind",
                    "time_in_nanos": 813581,
                    "breakdown": {
                      "set_min_competitive_score_count": 0,
                      "match_count": 0,
                      "shallow_advance_count": 0,
                      "set_min_competitive_score": 0,
                      "next_doc": 3291,
                      "match": 0,
                      "next_doc_count": 2,
                      "score_count": 2,
                      "compute_max_score_count": 0,
                      "compute_max_score": 0,
                      "advance": 7208,
                      "advance_count": 2,
                      "score": 18666,
                      "build_scorer_count": 6,
                      "create_weight": 616375,
                      "shallow_advance": 0,
                      "create_weight_count": 1,
                      "build_scorer": 168041
                    }
                  },
                  {
                    "type": "TermQuery",
                    "description": "title:rise",
                    "time_in_nanos": 191083,
                    "breakdown": {
                      "set_min_competitive_score_count": 0,
                      "match_count": 0,
                      "shallow_advance_count": 0,
                      "set_min_competitive_score": 0,
                      "next_doc": 0,
                      "match": 0,
                      "next_doc_count": 0,
                      "score_count": 0,
                      "compute_max_score_count": 0,
                      "compute_max_score": 0,
                      "advance": 0,
                      "advance_count": 0,
                      "score": 0,
                      "build_scorer_count": 2,
                      "create_weight": 188625,
                      "shallow_advance": 0,
                      "create_weight_count": 1,
                      "build_scorer": 2458
                    }
                  }
                ]
              }
            ],
            "rewrite_time": 192417,
            "collector": [
              {
                "name": "SimpleTopScoreDocCollector",
                "reason": "search_top_hits",
                "time_in_nanos": 77291
              }
            ]
          }
        ],
        "aggregations": []
      }
    ]
  }
}
```
</details>

## 响应字段

响应包括以下字段。

场地| 数据类型| 描述
:--- | :--- | :---
`profile` | 目的| 包含分析信息。
`profile.shards` | 对象数组| 可以针对索引中的一个或多个碎片执行搜索请求，搜索可能涉及一个或多个索引。就这样`profile.shards` 数组包含搜索中涉及的每个碎片的分析信息。
`profile.shards.id` | 细绳| 碎片的碎片ID`[node-ID][index-name][shard-ID]` 格式。
`profile.shards.searches` | 对象数组| 搜索代表针对基础Lucene索引执行的查询。大多数搜索请求都针对Lucene索引执行单个搜索，但是某些搜索请求可以执行多个搜索。例如，包括全局聚合会导致次要`match_all` 查询全球环境。这`profile.shards` 数组包含有关每个搜索执行的分析信息。
[`profile.shards.searches.query`](#the-query-array) | 对象数组| 分析有关查询执行的信息。
`profile.shards.searches.rewrite_time` | 整数| 所有Lucene疑问都是重写的。查询及其子女可能会多次重写，直到查询停止更改为止。重写过程涉及进行优化，例如删除冗余子句或更有效地替换查询路径。重写过程后，原始查询可能会发生重大变化。这`rewrite_time` 字段包含纳秒中查询及其所有子女的累积总重写时间。
[`profile.shards.searches.collector`](#the-collector-array) | 对象数组| 分析有关运行搜索的Lucene收集器的信息。
[`profile.shards.aggregations`](#aggregations) | 对象数组| 分析有关汇总执行的信息。

### 这`query` 大批

这`query` 数组包含具有以下字段的对象。

场地| 数据类型| 描述
:--- | :--- | :---
`type` | 细绳| 重写搜索查询的Lucene查询类型。对应于Lucene类名称（在OpenSearch中通常具有相同的名称）。
`description` | 细绳| 包含对查询的Lucene解释。帮助将查询与相同类型区分。
`time_in_nanos` | 长的| 查询在纳秒内执行的时间。在父母查询中，时间包括所有子女查询的执行时间。
[`breakdown`](#the-breakdown-object) | 目的| 包含有关低的定时统计数据-Level Lucene执行。
`children` | 对象数组| 如果查询具有子征服（儿童），则此字段包含有关子征服的信息。

### 这`breakdown` 目的

这`breakdown` 对象表示有关低的定时统计信息-将Lucene执行，按方法分解。时间在墙上列出-时钟纳米秒，未标准化。这`breakdown` 时机包括所有儿童时代。这`breakdown` 对象包括以下字段。所有字段都包含整数值。

场地| 描述
:--- | :--- 
`create_weight` | A`Query` Lucene中的物体是不变的。但是，露西恩应该能够重复使用`Query` 多个对象`IndexSearcher` 对象。因此，`Query` 对象需要保持与执行查询的索引相关的临时状态和统计信息。为了实现重复使用，每个`Query` 对象生成一个`Weight` 对象，该对象保持与临时上下文（状态）`<IndexSearcher, Query>` 元组。这`create_weight` 字段包含创建时间的时间`Weight` 目的。
`build_scorer` | A`Scorer` 迭代匹配文档，并为每个文档生成一个分数。这`build_scorer` 字段包含花费的时间生成`Scorer` 目的。这不包括对文档进行评分的时间。这`Scorer` 初始化时间取决于特定查询的优化和复杂性。这`build_scorer` 参数还包括与缓存相关的时间，如果用于查询并启用了缓存。
`next_doc` | 这`next_doc` Lucene方法返回与查询相匹配的下一个文档的文档ID。此方法是一种特殊类型`advance` 方法，等效于`advance(docId() + 1)`。这`next_doc` 对于许多Lucene查询，方法更方便。这`next_doc` 字段包含确定下一个匹配文档所需的时间，该文档取决于查询类型。
`advance` | 这`advance` 方法较低-级别版本的`next_doc` Lucene的方法。它还找到了下一个匹配的文档，但需要调用查询执行其他任务，例如识别跳过。一些查询，例如连词（`must` 布尔查询中的从句），无法使用`next_doc`。对于这些查询，`advance` 是定时的。
`match` | 对于某些查询，文档匹配以两个步骤执行。首先，该文档大约匹配。其次，通过更全面的过程对那些大约匹配的文档进行了检查。例如，首先检查文档是否包含短语中的所有术语。接下来，它验证了术语是否有序（这是一个更昂贵的过程）。这`match` 字段是非-仅适用于那些使用两个查询-步骤验证过程。
`score` | 包含花费的时间`Scorer` 为特定文档评分。
`shallow_advance` | 包含执行的时间`advanceShallow` Lucene方法。
`compute_max_score` | 包含执行的时间`getMaxScore` Lucene方法。
`set_min_competitive_score` | 包含执行的时间`setMinCompetitiveScore` Lucene方法。
`<method>_count` | 包含一个`<method>`。例如，`advance_count` 包含`advance` 方法。由于在不同的文档上调用该方法，因此发生了相同方法的不同调用。您可以通过比较不同查询组件中的计数来确定查询的选择性。

### 这`collector` 大批

这`collector` 阵列包含有关Lucene收集器的信息。收藏家负责协调文档遍历，评分和收集匹配文档。使用收集器，单个查询可以记录聚合结果并执行全局查询或发布-查询过滤器。

场地| 描述
:--- | :--- 
`name` | 收藏家名称。在里面[示例响应](#example-response)， 这`collector` 是一个`SimpleTopScoreDocCollector`---默认评分和分类收集器。
`reason` | 包含对收藏家的描述。对于可能的字段值，请参阅[收集者的原因](#collector-reasons)。
`time_in_nanos` | 一堵墙-时钟时间，包括所有儿童的时间。
`children` | 如果收集器具有子收集器（儿童），则此字段包含有关子策略器的信息。

收集器时间是独立计算，组合和归一化的，因此它们独立于查询时间。
{: .note}

#### 收集者的原因

下表描述了所有可用的收藏家原因。

原因| 描述
:--- | :--- 
`search_sorted` | 分数和分类文件的收藏家。在大多数简单的搜索中存在。
`search_count` | 计算匹配文档数量但不获取源的收集器。在当时`size: 0` 指定。
`search_terminate_after_count` | 一个收集器搜索匹配文档并在找到指定数量的文档时终止搜索。当时`terminate_after_count` 指定查询参数。
`search_min_score` | 一个收藏家返回其得分大于最低分数的匹配文档。当时`min_score` 参数已指定。
`search_multi` | 其他收藏家的包装收藏家。在单个搜索中将搜索，聚合，全局聚合和后过滤器组合在一起时存在。
`search_timeout` | 在指定的时间段内停止运行的收集器。在a`timeout` 参数已指定。
`aggregation` | 针对指定查询范围运行的聚合收集器。OpenSearch使用一个`aggregation` 收集器收集所有聚合的文件。
`global_aggregation` | 与全球查询范围相对的收藏家。全局范围与指定的查询范围不同，因此为了收集整个数据集`match_all` 查询必须运行。

## 聚合

要配置汇总，请发送聚合请求并提供`profile` 参数设置为`true`。

#### 示例请求：全局聚合

```json
GET /opensearch_dashboards_sample_data_ecommerce/_search
{
  "profile": "true",
  "size": 0,
  "query": {
    "match": { "manufacturer": "Elitelligence" }
  },
  "aggs": {
    "all_products": {
      "global": {}, 
      "aggs": {     
      "avg_price": { "avg": { "field": "taxful_total_price" } }
      }
    },
    "elitelligence_products": { "avg": { "field": "taxful_total_price" } }
  }
}
```
{% include copy-curl.html %}

#### 示例响应：全局聚合

响应包含分析信息：

<details closed markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}

```json
{
  "took": 10,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1370,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "all_products": {
      "doc_count": 4675,
      "avg_price": {
        "value": 75.05542864304813
      }
    },
    "elitelligence_products": {
      "value": 68.4430200729927
    }
  },
  "profile": {
    "shards": [
      {
        "id": "[LidyZ1HVS-u93-73Z49dQg][opensearch_dashboards_sample_data_ecommerce][0]",
        "inbound_network_time_in_millis": 0,
        "outbound_network_time_in_millis": 0,
        "searches": [
          {
            "query": [
              {
                "type": "ConstantScoreQuery",
                "description": "ConstantScore(manufacturer:elitelligence)",
                "time_in_nanos": 1367487,
                "breakdown": {
                  "set_min_competitive_score_count": 0,
                  "match_count": 0,
                  "shallow_advance_count": 0,
                  "set_min_competitive_score": 0,
                  "next_doc": 634321,
                  "match": 0,
                  "next_doc_count": 1370,
                  "score_count": 0,
                  "compute_max_score_count": 0,
                  "compute_max_score": 0,
                  "advance": 173250,
                  "advance_count": 2,
                  "score": 0,
                  "build_scorer_count": 4,
                  "create_weight": 132458,
                  "shallow_advance": 0,
                  "create_weight_count": 1,
                  "build_scorer": 427458
                },
                "children": [
                  {
                    "type": "TermQuery",
                    "description": "manufacturer:elitelligence",
                    "time_in_nanos": 1174794,
                    "breakdown": {
                      "set_min_competitive_score_count": 0,
                      "match_count": 0,
                      "shallow_advance_count": 0,
                      "set_min_competitive_score": 0,
                      "next_doc": 470918,
                      "match": 0,
                      "next_doc_count": 1370,
                      "score_count": 0,
                      "compute_max_score_count": 0,
                      "compute_max_score": 0,
                      "advance": 172084,
                      "advance_count": 2,
                      "score": 0,
                      "build_scorer_count": 4,
                      "create_weight": 114041,
                      "shallow_advance": 0,
                      "create_weight_count": 1,
                      "build_scorer": 417751
                    }
                  }
                ]
              }
            ],
            "rewrite_time": 42542,
            "collector": [
              {
                "name": "MultiCollector",
                "reason": "search_multi",
                "time_in_nanos": 778406,
                "children": [
                  {
                    "name": "EarlyTerminatingCollector",
                    "reason": "search_count",
                    "time_in_nanos": 70290
                  },
                  {
                    "name": "ProfilingAggregator: [elitelligence_products]",
                    "reason": "aggregation",
                    "time_in_nanos": 502780
                  }
                ]
              }
            ]
          },
          {
            "query": [
              {
                "type": "ConstantScoreQuery",
                "description": "ConstantScore(*:*)",
                "time_in_nanos": 995345,
                "breakdown": {
                  "set_min_competitive_score_count": 0,
                  "match_count": 0,
                  "shallow_advance_count": 0,
                  "set_min_competitive_score": 0,
                  "next_doc": 930803,
                  "match": 0,
                  "next_doc_count": 4675,
                  "score_count": 0,
                  "compute_max_score_count": 0,
                  "compute_max_score": 0,
                  "advance": 2209,
                  "advance_count": 2,
                  "score": 0,
                  "build_scorer_count": 4,
                  "create_weight": 23875,
                  "shallow_advance": 0,
                  "create_weight_count": 1,
                  "build_scorer": 38458
                },
                "children": [
                  {
                    "type": "MatchAllDocsQuery",
                    "description": "*:*",
                    "time_in_nanos": 431375,
                    "breakdown": {
                      "set_min_competitive_score_count": 0,
                      "match_count": 0,
                      "shallow_advance_count": 0,
                      "set_min_competitive_score": 0,
                      "next_doc": 389875,
                      "match": 0,
                      "next_doc_count": 4675,
                      "score_count": 0,
                      "compute_max_score_count": 0,
                      "compute_max_score": 0,
                      "advance": 1167,
                      "advance_count": 2,
                      "score": 0,
                      "build_scorer_count": 4,
                      "create_weight": 9458,
                      "shallow_advance": 0,
                      "create_weight_count": 1,
                      "build_scorer": 30875
                    }
                  }
                ]
              }
            ],
            "rewrite_time": 8792,
            "collector": [
              {
                "name": "ProfilingAggregator: [all_products]",
                "reason": "aggregation_global",
                "time_in_nanos": 1310536
              }
            ]
          }
        ],
        "aggregations": [
          {
            "type": "AvgAggregator",
            "description": "elitelligence_products",
            "time_in_nanos": 319918,
            "breakdown": {
              "reduce": 0,
              "post_collection_count": 1,
              "build_leaf_collector": 130709,
              "build_aggregation": 2709,
              "build_aggregation_count": 1,
              "build_leaf_collector_count": 2,
              "post_collection": 584,
              "initialize": 4750,
              "initialize_count": 1,
              "reduce_count": 0,
              "collect": 181166,
              "collect_count": 1370
            }
          },
          {
            "type": "GlobalAggregator",
            "description": "all_products",
            "time_in_nanos": 1519340,
            "breakdown": {
              "reduce": 0,
              "post_collection_count": 1,
              "build_leaf_collector": 134625,
              "build_aggregation": 59291,
              "build_aggregation_count": 1,
              "build_leaf_collector_count": 2,
              "post_collection": 5041,
              "initialize": 24500,
              "initialize_count": 1,
              "reduce_count": 0,
              "collect": 1295883,
              "collect_count": 4675
            },
            "children": [
              {
                "type": "AvgAggregator",
                "description": "avg_price",
                "time_in_nanos": 775967,
                "breakdown": {
                  "reduce": 0,
                  "post_collection_count": 1,
                  "build_leaf_collector": 98999,
                  "build_aggregation": 33083,
                  "build_aggregation_count": 1,
                  "build_leaf_collector_count": 2,
                  "post_collection": 2209,
                  "initialize": 1708,
                  "initialize_count": 1,
                  "reduce_count": 0,
                  "collect": 639968,
                  "collect_count": 4675
                }
              }
            ]
          }
        ]
      }
    ]
  }
}
```
</details>

#### 示例请求：非-全球聚合

```json
GET /opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "avg_taxful_total_price": {
      "avg": {
        "field": "taxful_total_price"
      }
    }
  }
}
```
{% include copy-curl.html %}

#### 示例响应：非-全球聚合

响应包含分析信息：

<details closed markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}

```json
{
  "took": 13,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4675,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "avg_taxful_total_price": {
      "value": 75.05542864304813
    }
  },
  "profile": {
    "shards": [
      {
        "id": "[LidyZ1HVS-u93-73Z49dQg][opensearch_dashboards_sample_data_ecommerce][0]",
        "inbound_network_time_in_millis": 0,
        "outbound_network_time_in_millis": 0,
        "searches": [
          {
            "query": [
              {
                "type": "ConstantScoreQuery",
                "description": "ConstantScore(*:*)",
                "time_in_nanos": 1690820,
                "breakdown": {
                  "set_min_competitive_score_count": 0,
                  "match_count": 0,
                  "shallow_advance_count": 0,
                  "set_min_competitive_score": 0,
                  "next_doc": 1614112,
                  "match": 0,
                  "next_doc_count": 4675,
                  "score_count": 0,
                  "compute_max_score_count": 0,
                  "compute_max_score": 0,
                  "advance": 2708,
                  "advance_count": 2,
                  "score": 0,
                  "build_scorer_count": 4,
                  "create_weight": 20250,
                  "shallow_advance": 0,
                  "create_weight_count": 1,
                  "build_scorer": 53750
                },
                "children": [
                  {
                    "type": "MatchAllDocsQuery",
                    "description": "*:*",
                    "time_in_nanos": 770902,
                    "breakdown": {
                      "set_min_competitive_score_count": 0,
                      "match_count": 0,
                      "shallow_advance_count": 0,
                      "set_min_competitive_score": 0,
                      "next_doc": 721943,
                      "match": 0,
                      "next_doc_count": 4675,
                      "score_count": 0,
                      "compute_max_score_count": 0,
                      "compute_max_score": 0,
                      "advance": 1042,
                      "advance_count": 2,
                      "score": 0,
                      "build_scorer_count": 4,
                      "create_weight": 5041,
                      "shallow_advance": 0,
                      "create_weight_count": 1,
                      "build_scorer": 42876
                    }
                  }
                ]
              }
            ],
            "rewrite_time": 22000,
            "collector": [
              {
                "name": "MultiCollector",
                "reason": "search_multi",
                "time_in_nanos": 3672676,
                "children": [
                  {
                    "name": "EarlyTerminatingCollector",
                    "reason": "search_count",
                    "time_in_nanos": 78626
                  },
                  {
                    "name": "ProfilingAggregator: [avg_taxful_total_price]",
                    "reason": "aggregation",
                    "time_in_nanos": 2834566
                  }
                ]
              }
            ]
          }
        ],
        "aggregations": [
          {
            "type": "AvgAggregator",
            "description": "avg_taxful_total_price",
            "time_in_nanos": 1973702,
            "breakdown": {
              "reduce": 0,
              "post_collection_count": 1,
              "build_leaf_collector": 199292,
              "build_aggregation": 13584,
              "build_aggregation_count": 1,
              "build_leaf_collector_count": 2,
              "post_collection": 6125,
              "initialize": 6916,
              "initialize_count": 1,
              "reduce_count": 0,
              "collect": 1747785,
              "collect_count": 4675
            }
          }
        ]
      }
    ]
  }
}
```
</details>

### 响应字段

这`aggregations` 数组包含带有以下字段的聚合对象。

场地| 数据类型| 描述
:--- | :--- | :---
`type` | 细绳| 聚合类型。在里面[非-全局聚合示例响应](#example-response-non-global-aggregation)，聚合类型是`AvgAggregator`。[全局聚合示例响应](#example-request-global-aggregation) 包含a`GlobalAggregator` 与`AvgAggregator` 孩子。
`description` | 细绳| 包含对聚集的Lucene解释。帮助将聚合与相同类型区分。
`time_in_nanos` | 长的| 在纳秒内执行聚合所花费的时间。在父母的汇总中，时间包括所有子女聚合的执行时间。
[`breakdown`](#the-breakdown-object-1) | 目的| 包含有关低的定时统计数据-Level Lucene执行。
`children` | 对象数组| 如果聚合具有亚物体（儿童），则此字段包含有关亚物种的信息。
`debug` | 目的| 一些聚合返回`debug` 描述基础执行细节的对象。

### 这`breakdown` 目的

这`breakdown` 对象表示有关低的定时统计信息-将Lucene执行，按方法分解。每个字段`breakdown` 对象表示在聚合中执行的内部Lucene方法。时间在墙上列出-时钟纳米秒，未标准化。这`breakdown` 时机包括所有儿童时代。这`breakdown` 对象由以下字段组成。所有字段都包含整数值。

场地| 描述
:--- | :--- 
`initialize` | 包含执行的时间`preCollection()` 回调方法`AggregationCollectorManager` 创建。
`build_leaf_collector`| 包含运行时间的时间`getLeafCollector()` 聚合的方法，该方法创建了一个新的收藏家来收集给定上下文。
`collect`| 包含将文档收集到水桶中花费的时间。
`post_collection`| 包含运行聚合的时间`postCollection()` 回调方法。
`build_aggregation`| 包含运行聚合的时间`buildAggregations()` 方法，它构建了此聚合的结果。
`reduce`| 包含在`reduce` 阶段。
`<method>_count` | 包含一个`<method>`。例如，`build_leaf_collector_count` 包含`build_leaf_collector` 方法。

## 并发段搜索

从OpenSearch 2.10开始[并发段搜索]({{site.url}}{{site.baseurl}}/search-plugins/concurrent-segment-search/) 允许每个碎片-在查询阶段并行搜索段的级别请求。如果启用实验并发段搜索功能标志，则配置文件API响应将包含一些其他字段，其中包含有关_slices_的统计信息。

切片是可以由线程执行的工作单位。每个查询都可以分为多个切片，每个切片包含一个或多个段。所有切片可以根据池中的可用线程并行或某些顺序执行。

通常，最大/分钟/AVG切片时间捕获了所有切片的统计信息，以达到计时类型。例如，在分析聚合时`max_slice_time_in_nanos` 字段`aggregations` 部分显示了汇总操作及其在所有切片中的孩子所消耗的最长时间。

#### 示例响应

以下是与三个片段切片的并发搜索的示例响应：

<details closed markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}

```json
{
  "took": 76,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 5,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      ...
    ]
  },
  "aggregations": {
    ...
  },
  "profile": {
    "shards": [
      {
        "id": "[Sn2zHhcMTRetEjXvppU8bA][idx][0]",
        "inbound_network_time_in_millis": 0,
        "outbound_network_time_in_millis": 0,
        "searches": [
          {
            "query": [
              {
                "type": "MatchAllDocsQuery",
                "description": "*:*",
                "time_in_nanos": 429246,
                "breakdown": {
                  "set_min_competitive_score_count": 0,
                  "match_count": 0,
                  "shallow_advance_count": 0,
                  "set_min_competitive_score": 0,
                  "next_doc": 5485,
                  "match": 0,
                  "next_doc_count": 5,
                  "score_count": 5,
                  "compute_max_score_count": 0,
                  "compute_max_score": 0,
                  "advance": 3350,
                  "advance_count": 3,
                  "score": 5920,
                  "build_scorer_count": 6,
                  "create_weight": 429246,
                  "shallow_advance": 0,
                  "create_weight_count": 1,
                  "build_scorer": 2221054
                }
              }
            ],
            "rewrite_time": 12442,
            "collector": [
              {
                "name": "QueryCollectorManager",
                "reason": "search_multi",
                "time_in_nanos": 6786930,
                "reduce_time_in_nanos": 5892759,
                "max_slice_time_in_nanos": 5951808,
                "min_slice_time_in_nanos": 5798174,
                "avg_slice_time_in_nanos": 5876588,
                "slice_count": 3,
                "children": [
                  {
                    "name": "SimpleTopDocsCollectorManager",
                    "reason": "search_top_hits",
                    "time_in_nanos": 1340186,
                    "reduce_time_in_nanos": 1084060,
                    "max_slice_time_in_nanos": 457165,
                    "min_slice_time_in_nanos": 433706,
                    "avg_slice_time_in_nanos": 443332,
                    "slice_count": 3
                  },
                  {
                    "name": "NonGlobalAggCollectorManager: [histo]",
                    "reason": "aggregation",
                    "time_in_nanos": 5366791,
                    "reduce_time_in_nanos": 4637260,
                    "max_slice_time_in_nanos": 4526680,
                    "min_slice_time_in_nanos": 4414049,
                    "avg_slice_time_in_nanos": 4487122,
                    "slice_count": 3
                  }
                ]
              }
            ]
          }
        ],
        "aggregations": [
          {
            "type": "NumericHistogramAggregator",
            "description": "histo",
            "time_in_nanos": 16454372,
            "max_slice_time_in_nanos": 7342096,
            "min_slice_time_in_nanos": 4413728,
            "avg_slice_time_in_nanos": 5430066,
            "breakdown": {
              "min_build_leaf_collector": 4320259,
              "build_aggregation_count": 3,
              "post_collection": 9942,
              "max_collect_count": 2,
              "initialize_count": 3,
              "reduce_count": 0,
              "avg_collect": 146319,
              "max_build_aggregation": 2826399,
              "avg_collect_count": 1,
              "max_build_leaf_collector": 4322299,
              "min_build_leaf_collector_count": 1,
              "build_aggregation": 3038635,
              "min_initialize": 1057,
              "max_reduce": 0,
              "build_leaf_collector_count": 3,
              "avg_reduce": 0,
              "min_collect_count": 1,
              "avg_build_leaf_collector_count": 1,
              "avg_build_leaf_collector": 4321197,
              "max_collect": 181266,
              "reduce": 0,
              "avg_build_aggregation": 954896,
              "min_post_collection": 1236,
              "max_initialize": 11603,
              "max_post_collection": 5350,
              "collect_count": 5,
              "avg_post_collection": 2793,
              "avg_initialize": 4860,
              "post_collection_count": 3,
              "build_leaf_collector": 4322299,
              "min_collect": 78519,
              "min_build_aggregation": 8543,
              "initialize": 11971068,
              "max_build_leaf_collector_count": 1,
              "min_reduce": 0,
              "collect": 181838
            },
            "debug": {
              "total_buckets": 1
            }
          }
        ]
      }
    ]
  }
}
```
</details>

### 修改或添加的响应字段

以下各节包含所有修改或添加的响应字段的定义，用于并发段搜索。

#### 这`query` 大批

|场地|描述|
|:---	|:---	|
|`time_in_nanos`|用于并发段搜索，`time_in_nanos` 是在纳秒中运行所有切片的所有方法所花费的累积时间。这不等于查询所花费的实际时间，因为它没有考虑到多个切片可以并行运行该方法。|
|`breakdown.<method>`|对于并发段搜索，此字段包含所有段运行方法所花费的时间。|
|`breakdown.<method>_count`|对于并发段搜索，此字段包含一个调用总数`<method>` 通过添加所有段的方法调用数量获得。|

#### 这`collector` 大批

|场地|描述|
|:---	|:---	|
|`time_in_nanos`|该收集器的总时间为纳秒。用于并发段搜索，`time_in_nanos` 是所有切片的总时间（`max(slice_end_time) - min(slice_start_time)`）。|
|`max_slice_time_in_nanos`|任何切片在纳秒中花费的最长时间。|
|`min_slice_time_in_nanos`|任何切片在纳秒中花费的最短时间。|
|`avg_slice_time_in_nanos`|任何切片的平均时间，纳米秒。|
|`slice_count`|此查询的总切片计数。|
|`reduce_time_in_nanos`|减少纳米秒的所有切片收集器的结果所花费的时间。|

#### 这`aggregations` 大批

|场地|描述|
|:---	|:---	|
|`time_in_nanos`|该聚合的总时间是纳秒。用于并发段搜索，`time_in_nanos` 是所有切片的总时间（`max(slice_end_time) - min(slice_start_time)`）。|
|`max_slice_time_in_nanos`|任何切片在纳秒中运行聚合所花费的最长时间。|
|`min_slice_time_in_nanos`|任何切片在纳秒中运行聚合所花费的最短时间。|
|`avg_slice_time_in_nanos`|任何切片在纳秒中运行聚合的平均时间。|
|`<method>`|所有切片的总过去时间（`max(slice_end_time) - min(slice_start_time)`）。例如，对于`collect` 方法，这是将文档收集到所有切片中的水桶中花费的总时间。|
|`max_<method>`|任何切片所花费的最长时间运行聚合方法。|
|`min_<method>`|任何切片所花费的最短时间运行聚合方法。|
|`avg_<method>`|任何切片所需的平均时间运行聚合方法。|
|`<method>_count`|总方法在所有切片中计数。例如，对于`collect` 方法，这是此方法的总数，将文档收集到所有切片的存储桶中所需的总数。|
|`max_<method>_count`|A的最大召唤数量`<method>` 在任何切片上。|
|`min_<method>_count`|最小数量`<method>` 在任何切片上。|
|`avg_<method>_count`|A的平均调用数量`<method>` 在任何切片上。|


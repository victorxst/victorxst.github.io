---
layout: default
title: 搜索
nav_order: 75
redirect_from:
  - /opensearch/rest-api/search/
---

# 搜索
**引入1.0**
{: .label .label-purple }

搜索API操作使您可以执行搜索请求，以搜索群集以获取数据。

## 例子

```json
GET /movies/_search
{
  "query": {
    "match": {
      "text_entry": "I am the night"
    }
  }
}
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
GET /<target-index>/_search
GET /_search

POST /<target-index>/_search
POST /_search
```

## URL参数

所有URL参数都是可选的。

范围| 类型| 描述
:--- | :--- | :---
允许_no_indices| 布尔| 是否忽略不符合任何索引的通配符。默认是正确的。
laster_partial_search_results| 布尔| 如果请求陷入错误或时，是否要返回部分结果。默认是正确的。
分析仪| 细绳| 分析仪用于查询字符串。
Analyze_wildCard| 布尔| 更新操作是否应在分析中包括通配符和前缀查询。默认值为false。
batched_reduce_size| 整数| 在节点上减少多少碎片结果。默认值为512。
cancel_after_time_interval| 时间| 之后将取消搜索请求的时间。要求-级别参数优先于cancel_after_time_interval[集群设置]({{site.url}}{{site.baseurl}}/api-reference/cluster-settings)。默认为-1。
CCS_MINIMIZE_ROUNDTRIPS| 布尔| 是否最小化节点和远程簇之间的往返。默认是正确的。
Default_operator| 细绳| 指示字符串查询的默认运算符应该是和或或。默认为或。
DF| 细绳| 如果查询字符串中未提供字段前缀，则默认字段。
docvalue_fields| 细绳| OpenSearch的字段应使用其DocValue表单返回。
Expand_WildCard| 细绳| 指定通配符表达式可以匹配的索引类型。支持逗号-分离的值。有效值全部（匹配任何索引），打开（匹配打开，非-隐藏索引），封闭（匹配封闭，非-隐藏索引），隐藏（匹配隐藏索引）和无（拒绝通配符表达式）。默认值打开。
解释| 布尔| 是否返回有关OpenSearch如何计算文档分数的详细信息。默认值为false。
从| 整数| 搜索的起始索引。默认值为0。
ignore_throttled| 布尔| 如果冻结索引，是否忽略混凝土，扩展或具有别名的索引。默认是正确的。
ignore_unavailable| 布尔| 指定响应中是否包括缺失或封闭索引。默认值为false。
宽容| 布尔| 指定如果查询具有格式错误，OpenSearch是否应接受请求（例如，查询文本字段中的整数）。默认值为false。
max_concurrent_shard_requests| 整数| 该请求应在每个节点上执行多少并发碎片请求。默认值为5。
phase_took| 布尔| 是否返回阶段-等级`took` 响应中的时间值。默认值为false。
pre_filter_shard_size| 整数| 如果请求超过阈值，则触发前滤器操作的较大尺寸阈值。默认值为128个碎片。
偏爱| 细绳| 指定OpenSearch应执行搜索的碎片或节点。对于有效的值，请参阅[这`preference` 查询参数](#the-preference-query-parameter)。
问| 细绳| Lucene查询字符串的查询。
request_cache| 布尔| 指定OpenSearch是否应使用请求缓存。默认值是是否在索引的设置中启用。
REST_TOTAL_HITS_AS_INT| 布尔| 是否返回`hits.total` 作为整数。否则返回对象。默认值为false。
路由| 细绳| 用查询操作将更新路由到特定碎片的值。
滚动| 时间| 保持搜索上下文开放多长时间。
搜索类型| 细绳| 计算相关性分数时，OpenSearch是否应使用全球术语和文档频率。有效的选择是`query_then_fetch` 和`dfs_query_then_fetch`。`query_then_fetch` 使用本地术语和文档频率为碎片评分文档。通常更快但准确。`dfs_query_then_fetch` 使用全球术语和文档频率在所有碎片中得分。通常较慢，但更准确。默认为`query_then_fetch`。
seq_no_primary_term| 布尔| 是否要返回每个文档命中最后一个操作的序列号和主要项。
尺寸| 整数| 响应中要包含多少结果。
种类| 列表| 逗号-＆lt; field＆gt;：＆lt; Direction＆gt;对分类。
_来源| 细绳| 是否包括`_source` 响应中的字段。
_source_excludes| 列表| 逗号-分开的源字段列表以外的响应中排除。
_source_includes| 列表| 逗号-分开的源字段列表要包含在响应中。
统计| 细绳| 与其他日志记录请求相关联的价值。
stored_fields| 布尔| Get操作是否应检索存储在索引中的字段。默认值为false。
建议_FIELD| 细绳| Fields Opensearch可以用来寻找类似的术语。
建议_mode| 细绳| 搜索时使用的模式。可用选项是`always` （根据提供的条款使用建议），`popular` （使用更多发生的建议），并且`missing` （使用索引中的术语使用建议）。
建议_size| 整数| 有多少个建议要返回。
建议_text| 细绳| 建议应基于建议的来源。
terminate_after| 整数| 文档数量的最大数量OpenSearch应在终止请求之前进行处理。默认值为0。
暂停| 时间| 操作应等待主动碎片的响应多长时间。默认为`1m`。
track_scores| 布尔| 是否返回文档分数。默认值为false。
track_total_hits| 布尔或整数| 是否返回匹配查询的多少个文档。
typed_keys| 布尔| 返回的汇总和建议的条款是否应包括其类型。默认是正确的。
版本| 布尔| 是否将文档版本作为匹配项。

### 这`preference` 查询参数

这`preference` 查询参数指定OpenSearch应执行搜索的碎片或节点。以下是有效值：

- `_primary`：仅在主碎片上执行搜索。
- `_replica`：仅在复制碎片上执行搜索。
- `_primary_first`：在主碎片上执行搜索，但如果没有主碎片，则会失败其他可用的碎片。
- `_replica_first`：在副本片上执行搜索，但如果没有复制碎片，则会失败其他可用的碎片。
- `_local`：如果可能，请在本地节点的碎片上执行搜索。
- `_prefer_nodes:<node-id-1>,<node-id-2>`：如果可能，请在指定的节点上执行搜索。使用逗号-分开列表以指定多个节点。
- `_shards:<shard-id-1>,<shard-id-2>`：仅在指定的碎片上执行搜索。使用逗号-分开列表以指定多个碎片。与其他偏好结合在一起时`_shards` 首选必须首先列出。例如，`_shards:1,2|_replica`。
- `_only_nodes:<node-id-1>,<node-id-2>`：仅在指定的节点上执行搜索。使用逗号-分开列表以指定多个节点。
- `<string>`：指定用于搜索的自定义字符串。字符串不能以下划线开始（`_`）。具有相同自定义字符串的搜索被路由到同一碎片。

## 请求身体

所有字段都是可选的。

场地| 类型| 描述
:--- | :--- | :---
docvalue_fields| 对象数组| OpenSearch的字段应使用其DocValue表单返回。指定格式以返回某种格式的结果，例如日期和时间。
字段| 大批| 在请求中搜索的字段。指定格式以返回某种格式的结果，例如日期和时间。
解释| 细绳| 是否返回有关OpenSearch如何计算文档分数的详细信息。默认值为false。
从| 整数| 搜索的起始索引。默认值为0。
indices_boost| 对象数组| 用于提高指定索引的分数的值。以＆lt; index＆gt;的格式指定：＆lt; Boost-乘数＆gt;
min_score| 整数| 指定一个分数阈值以仅返回高于阈值的文档。
询问| 目的| 这[DSL查询]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/index) 在请求中使用。
seq_no_primary_term| 布尔| 是否要返回每个文档命中最后一个操作的序列号和主要项。
尺寸| 整数| 返回多少结果。默认值为10。
_来源| | 是否包括`_source` 响应中的字段。
统计| 细绳| 与其他日志记录请求相关联的价值。
terminate_after| 整数| 文档数量的最大数量OpenSearch应在终止请求之前进行处理。默认值为0。
暂停| 时间| 等待回复多长时间。默认值不是超时。
版本| 布尔| 是否将文档版本包括在响应中。

## 响应主体

```json
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [
      {
        "_index": "superheroes",
        "_id": "1",
        "_score": 1.0,
        "_source": {
          "superheroes": [
            {
              "Hero name": "Superman",
              "Real identity": "Clark Kent",
              "Age": 28
            },
            {
              "Hero name": "Batman",
              "Real identity": "Bruce Wayne",
              "Age": 26
            },
            {
              "Hero name": "Flash",
              "Real identity": "Barry Allen",
              "Age": 28
            },
            {
              "Hero name": "Robin",
              "Real identity": "Dick Grayson",
              "Age": 15
            }
          ]
        }
      }
    ]
  }
}
```

## 这`ext` 目的

从OpenSearch 2.10开始，插件作者可以添加`ext` 目的是搜索响应。目的`ext` 对象是包含插件-特定响应字段。例如，在对话搜索中，检索增强生成（RAG）的结果是一个"hit" （回答）。插件作者可以将此答案包括在搜索响应中`ext` 对象使其与搜索命中分开。在以下示例响应中，抹布结果在`ext.retrieval_augmented_generation.answer` 场地：

```json
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 3,
    "successful": 3,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 110,
      "relation": "eq"
    },
    "max_score": 0.55129033,
    "hits": [
      {
       "_index": "...",
        "_id": "...",
        "_score": 0.55129033,
        "_source": {
          "text": "...",
          "title": "..."
        }
      },
      {
      ...
      }
      ...
      {
      ...
      }
    ],
  }, // end of hits
  "ext": {
    "retrieval_augmented_generation": { // a search response processor
      "answer": "RAG answer"
    }
  }
} 
```


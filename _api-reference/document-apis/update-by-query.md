---
layout: default
title: 通过查询更新
parent: 文档API
nav_order: 50
redirect_from: 
 - /opensearch/rest-api/document-apis/update-by-query/
---

# 通过查询更新
**引入1.0**
{: .label .label-purple}

您可以将查询和脚本包含在您的更新请求的一部分中，因此OpenSearch可以运行脚本以更新与查询匹配的所有文档。

## 例子

```json
POST test-index1/_update_by_query
{
  "query": {
    "term": {
      "oldValue": 10
    }
  },
  "script" : {
    "source": "ctx._source.oldValue += params.newValue",
    "lang": "painless",
    "params" : {
      "newValue" : 20
    }
  }
}
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
POST <target-index1>, <target-index2>/_update_by_query
```

## URL参数

所有URL参数都是可选的。

范围| 类型| 描述
:--- | :--- | :--- | :---
＆lt; index＆gt;| 细绳| 逗号-分开的索引列表要更新。要更新所有索引，请使用 *或省略此参数。
允许_no_indices| 布尔| 是否忽略不符合任何索引的通配符。默认为`true`。
分析仪| 细绳| 分析仪用于查询字符串。
Analyze_wildCard| 布尔| 更新操作是否应在分析中包括通配符和前缀查询。默认值为false。
冲突| 细绳| 指示开放搜索，如果通过查询操作进行更新将发生在版本冲突中，该怎么办。有效的选项是`abort` 和`proceed`。默认为`abort`。
Default_operator| 细绳| 指示字符串查询的默认运算符应该是`AND` 或者`OR`。默认为`OR`。
DF| 细绳| 如果查询字符串中未提供字段前缀，则默认字段。
Expand_WildCard| 细绳| 指定通配符表达式可以匹配的索引类型。支持逗号-分离的值。有效值是`all` （匹配任何索引），`open` （匹配开放，非-隐藏索引），`closed` （匹配封闭，非-隐藏索引），`hidden` （匹配隐藏索引），`none` （拒绝通配符表达）。默认为`open`。
从| 整数| 搜索的起始索引。默认值为0。
ignore_unavailable| 布尔| 是在响应中排除丢失还是封闭索引。默认值为false。
宽容| 布尔| 指定如果查询具有格式错误，OpenSearch是否应接受请求（例如，查询文本字段中的整数）。默认值为false。
max_docs| 整数| 查询操作更新的更新最多应处理多少文档。默认是所有文档。
管道| 细绳| 用于处理文档的管道的ID。
偏爱| 细绳| 指定哪些碎片或节点OpenSearch应通过查询操作执行更新。
问| 细绳| Lucene查询字符串的查询。
request_cache| 布尔| 指定OpenSearch是否应使用请求缓存。默认值是是否在索引的设置中启用。
刷新| 布尔| 如果为true，则OpenSearch刷新碎片以通过查询操作进行更新以搜索结果。有效的选项是`true` 和`false`。默认为`false`。
requests_per_second| 整数| 指定请求在sub中的节流-每秒请求。默认为-1，这意味着没有节流。
路由| 细绳| 用查询操作将更新路由到特定碎片的值。
滚动| 时间| 保持搜索上下文开放多长时间。
scroll_size| 整数| 操作滚动请求的大小。默认值为1000。
搜索类型| 细绳| OpenSearch是否应使用全球术语和记录计算相关性分数的频率。有效的选择是`query_then_fetch` 和`dfs_query_then_fetch`。`query_then_fetch` 使用本地术语和文档频率为碎片评分文档。通常更快但准确。`dfs_query_then_fetch` 使用全球术语和文档频率在所有碎片中得分。通常较慢，但更准确。默认为`query_then_fetch`。
search_timeout| 时间| 等到OpenSearch认为该请求的时间超时。默认值不是超时。
切片| 字符串或整数| 将操作分为更快处理的数字切片，由整数指定。设置为`auto` OpenSearch应该决定操作的切片数量。默认为`1`，这表明操作不会分开。
种类| 列表| 逗号-＆lt; field＆gt;：＆lt; Direction＆gt;对分类。
_来源| 细绳| 是否包括`_source` 响应中的字段。
_source_excludes| 细绳| 逗号-分开的源字段列表以外的响应中排除。
_source_includes| 细绳| 逗号-分开的源字段列表要包含在响应中。
统计| 细绳| 与其他日志记录请求相关联的价值。
terminate_after| 整数| 文档数量的最大数量OpenSearch应在终止请求之前进行处理。
暂停| 时间| 操作应从活动碎片的响应中等待多长时间。默认为`1m`。
版本| 布尔| 是否将文档版本作为匹配项。
wait_for_active_shards| 细绳| 在OpenSearch执行操作之前必须活跃的碎片数。有效值是`all` 或索引中碎片总数的任何整数。默认值为1，这是主要碎片。
wait_for_completion| 布尔| 设置为`false`，响应主体包括一个任务ID，OpenSearch异步执行操作。任务ID可用于检查任务的状态或取消任务。默认设置为`true`。

## 请求身体

要通过查询更新您的索引和文档，您必须包括一个[询问]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/index) OpenSearch可以运行的请求主体中的脚本可以更新您的文档。如果您没有指定查询，则索引中的每个文档都会更新。

```json
{
  "query": {
    "term": {
      "oldValue": 20
    }
  },
  "script" : {
    "source": "ctx._source.oldValue += params.newValue",
    "lang": "painless",
    "params" : {
      "newValue" : 10
    }
  }
}
```

## 回复
```json
{
  "took": 21,
  "timed_out": false,
  "total": 1,
  "updated": 1,
  "deleted": 0,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1.0,
  "throttled_until_millis": 0,
  "failures": []
}
```

## 响应身体场

场地| 描述
:--- | :---
拿| 完成操作所需的毫秒搜索时间。
时间到| 操作期间是否有任何更新请求。
全部的| 处理的文档总数。
更新| 已更新的文档总数。
批次| 滚动响应数量已处理的请求。
version_conflicts| 请求遇到的冲突数量。
零| 操作过程中忽略了多少个更新请求。该字段总是返回0。
重试| 批量和搜索重试请求的数量。
throttled_millis| 在请求期间，毫秒的毫秒数。
requests_per_second| 操作过程中每秒执行的请求数。
throttled_until_millis| OpenSearch执行下一个节流请求之前的时间。在查询请求的更新中，始终等于0。
失败| 请求期间发生的任何故障。


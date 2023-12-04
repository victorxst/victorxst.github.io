---
layout: default
title: 通过查询删除
parent: 文档API
nav_order: 40
redirect_from:
 - /opensearch/rest-api/document-apis/delete-by-query/
---

# 通过查询删除
**引入1.0**
{: .label .label-purple}

您可以在删除请求的一部分中包含查询，因此OpenSearch删除所有匹配该查询的文档。

## 例子

```json
POST sample-index1/_delete_by_query
{
  "query": {
    "match": {
      "movie-length": "124"
    }
  }
}
```
{% include copy-curl.html %}

## 路径和HTTP方法

```
POST <index>/_delete_by_query
```

## URL参数

所有URL参数都是可选的。

范围| 类型| 描述
:--- | :--- | :--- | :---
＆lt; index＆gt;| 细绳| 数据流，索引或别名的名称或列表要从中删除。支持通配符。如果剩下空白，OpenSearch搜索所有索引。
允许_no_indices| 布尔| 是否忽略不符合任何索引的通配符。默认为`true`。
分析仪| 细绳| 用于查询字符串中的分析仪。
Analyze_wildCard| 布尔| 指定是否分析通配符和前缀查询。默认值为false。
冲突| 细绳| 指示开放搜索，如果通过查询操作删除将发生什么会发生什么，则会发生在版本冲突中。有效的选项是`abort` 和`proceed`。默认为`abort`。
Default_operator| 细绳| 指示字符串查询的默认运算符应该是和或或。默认为或。
DF| 细绳| 如果查询字符串中未提供字段前缀，则默认字段。
Expand_WildCard| 细绳| 指定通配符表达式可以匹配的索引类型。支持逗号-分离的值。有效值是`all` （匹配任何索引），`open` （匹配开放，非-隐藏索引），`closed` （匹配封闭，非-隐藏索引），`hidden` （匹配隐藏索引），`none` （拒绝通配符表达）。默认为`open`。
从| 整数| 搜索的起始索引。默认值为0。
ignore_unavailable| 布尔| 指定响应中是否包括缺失或封闭索引。默认值为false。
宽容| 布尔| 指定如果查询具有格式错误，OpenSearch是否应接受请求（例如，查询文本字段中的整数）。默认值为false。
max_docs| 整数| 查询操作删除删除的文档最多应处理。默认是所有文档。
偏爱| 细绳| 指定哪些碎片或节点OpenSearch应通过查询操作执行删除。
问| 细绳| Lucene查询字符串的查询。
request_cache| 布尔| 指定OpenSearch是否应使用请求缓存。默认值是是否在索引的设置中启用。
刷新| 布尔| 如果为true，则OpenSearch刷新碎片以通过查询操作删除可用于搜索结果。有效的选项是`true`，`false`， 和`wait_for`，它告诉Opensearch在执行操作之前等待刷新。默认为`false`。
requests_per_second| 整数| 指定请求在sub中的节流-每秒请求。默认为-1，这意味着没有节流。
路由| 细绳| 用于将操作路由到特定碎片的值。
滚动| 时间| 搜索上下文应打开的时间。
scroll_size| 整数| 操作滚动请求的大小。默认值为1000。
搜索类型| 细绳| OpenSearch是否应使用全球术语和记录计算启示分数的频率。有效的选择是`query_then_fetch` 和`dfs_query_then_fetch`。`query_then_fetch` 使用本地术语和文档频率为碎片评分文档。通常更快但准确。`dfs_query_then_fetch` 使用全球术语和文档频率在所有碎片中得分。通常较慢，但更准确。默认为`query_then_fetch`。
search_timeout| 时间| 等到OpenSearch认为该请求的时间超时。默认值不是超时。
切片| 字符串或整数| 有多少个切片可以将操作切成更快的处理。指定整数设置多少个切片以将操作分为或使用`auto`，它告诉OpenSearch，它应该决定要分成多少个切片。如果您的索引中有很多碎片，请设置较低的数字以提高效率。默认值为1，这意味着任务不应划分。
种类| 细绳| 逗号-＆lt; field＆gt;：＆lt; Direction＆gt;对分类。
_来源| 细绳| 指定是否包括`_source` 响应中的字段。
_source_excludes| 细绳| 逗号-分开的源字段列表以外的响应中排除。
_source_includes| 细绳| 逗号-分开的源字段列表要包含在响应中。
统计| 细绳| 与其他日志记录请求相关联的价值。
terminate_after| 整数| 文档数量的最大数量OpenSearch应在终止请求之前进行处理。
暂停| 时间| 操作应从活动碎片的响应中等待多长时间。默认为`1m`。
版本| 布尔| 是否将文档版本作为匹配项。
wait_for_active_shards| 细绳| 在OpenSearch执行操作之前必须活跃的碎片数。有效值是`all` 或索引中碎片总数的任何整数。默认值为1，这是主要碎片。
wait_for_completion| 布尔| 将此参数设置为false指示opensearch它不应等待完成，也不应等高地执行此请求。异步请求在后台运行，您可以使用[任务]({{site.url}}{{site.baseurl}}/api-reference/tasks) API监视进度。


## 请求身体

要搜索您的索引以查看特定文档，您必须包括一个[询问]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/index) 在OpenSearch用来匹配文档的请求主体中。如果您不使用查询，OpenSearch将您的删除请求视为简单[删除文档操作]({{site.url}}{{site.baseurl}}/api-reference/document-apis/delete-document)。

```json
{
  "query": {
    "match": {
      "movie-length": "124"
    }
  }
}
```

## 回复
```json
{
  "took": 143,
  "timed_out": false,
  "total": 1,
  "deleted": 1,
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
时间到| 在操作时是否有任何删除请求。
全部的| 处理的文档总数。
删除| 删除的文档总数。
批次| 滚动响应数量已处理的请求。
version_conflicts| 请求遇到的冲突数量。
零| 在操作过程中忽略了多少删除请求。该字段总是返回0。
重试| 批量和搜索重试请求的数量。
throttled_millis| 在请求期间，毫秒的毫秒数。
requests_per_second| 操作过程中每秒执行的请求数。
throttled_until_millis| OpenSearch执行下一个节流请求之前的时间。通过查询请求始终等于删除中的0。
失败| 请求期间发生的任何故障。


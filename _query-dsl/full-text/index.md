---
layout: default
title: 全文查询
has_children: true
has_toc: false
nav_order: 30
redirect_from:
  - /opensearch/query-dsl/full-text/
  - /opensearch/query-dsl/full-text/index/
  - /query-dsl/query-dsl/full-text/
  - /query-dsl/full-text/
---

# 全文查询

此页面列出了全部-文本查询类型和常见选项。您可以使用许多可选字段来创建微妙的搜索行为，因此我们建议您在使用多个选项执行更高级或更复杂的搜索之前测试一些针对代表性索引的基本查询类型并验证输出。

OpenSearch使用Apache Lucene搜索库，该库提供了高效的数据结构和算法，用于摄入，索引，搜索和汇总数据。

要了解有关搜索查询类的更多信息，请参阅[Lucene Query Javadocs](https://lucene.apache.org/core/8_9_0/core/org/apache/lucene/search/Query.html)。

完整-本节中显示的文本查询类型使用标准分析仪，该标准分析仪会在提交查询时自动分析文本。

下表列出了全部-文本查询类型。

查询类型| 描述
:--- | :--- 
[`intervals`]({{site.url}}{{site.baseurl}}/query-dsl/full-text/intervals/) | 允许-对匹配术语的近距离和顺序的粒度控制。
[`match`]({{site.url}}{{site.baseurl}}/query-dsl/full-text/match/) | 默认完整-文本查询，可用于模糊匹配，短语或接近搜索。
[`match_bool_prefix`]({{site.url}}{{site.baseurl}}/query-dsl/full-text/match-bool-prefix/) | 创建一个[布尔查询]({{site.url}}{{site.baseurl}}/query-dsl/compound/bool/) 这与任何位置的所有术语匹配，将最后一个术语视为前缀。
[`match_phrase`]({{site.url}}{{site.baseurl}}/query-dsl/full-text/match-phrase/) | 类似于`match` 查询，但将整个短语与可配置的斜率匹配。
[`match_phrase_prefix`]({{site.url}}{{site.baseurl}}/query-dsl/full-text/match-phrase-prefix/) | 类似于`match_phrase` 查询但将术语匹配为整个短语，将最后一个术语视为前缀。
[`multi_match`]({{site.url}}{{site.baseurl}}/query-dsl/full-text/multi-match/) | 类似于`match` 查询，但用于多个字段。
[`query_string`]({{site.url}}{{site.baseurl}}/query-dsl/full-text/query-string/) | 使用严格的语法来指定布尔条件和多种条件-单个查询字符串中的现场搜索。
[`simple_query_string`]({{site.url}}{{site.baseurl}}/query-dsl/full-text/simple-query-string/) | 更简单，更严格的版本`query_string` 询问。


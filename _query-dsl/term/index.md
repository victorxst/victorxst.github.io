---
layout: default
title: 术语级查询
has_children: true
has_toc: false
nav_order: 20
---

# 术语级查询

学期-Level查询搜索包含精确搜索词的文档的索引。根据期限返回的文件-级别查询不是通过其相关性分数来排序的。

使用文本数据时，使用术语-映射为字段的级别查询`keyword` 仅有的。

学期-级别查询不适合搜索分析的文本字段。要返回分析的字段，请使用[满的-文本查询]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/full-text/)。

## 术语级查询类型

下表列出了所有术语-级查询类型。

查询类型| 描述
:--- | :--- 
[`term`]({{site.url}}{{site.baseurl}}/query-dsl/term/term/) | 搜索包含特定字段中确切术语的文档。
[`terms`]({{site.url}}{{site.baseurl}}/query-dsl/term/terms/) | 搜索在特定字段中包含一个或多个术语的文档。
[`terms_set`]({{site.url}}{{site.baseurl}}/query-dsl/term/terms-set/) | 搜索与特定字段中最少的条款数量匹配的文档。
[`ids`]({{site.url}}{{site.baseurl}}/query-dsl/term/ids/) | 通过文档ID搜索文档。
[`range`]({{site.url}}{{site.baseurl}}/query-dsl/term/range/) | 搜索具有特定范围内字段值的文档。
[`prefix`]({{site.url}}{{site.baseurl}}/query-dsl/term/prefix/) | 搜索包含以特定前缀开头的术语的文档。
[`exists`]({{site.url}}{{site.baseurl}}/query-dsl/term/exists/) | 在特定字段中搜索具有任何索引值的文档。
[`fuzzy`]({{site.url}}{{site.baseurl}}/query-dsl/term/fuzzy/) | 搜索包含与搜索词相似的术语的文档[Levenshtein距离](https://en.wikipedia.org/wiki/Levenshtein_distance)。Levenshtein距离测量一个-将一个术语更改为另一个术语所需的角色更改。
[`wildcard`]({{site.url}}{{site.baseurl}}/query-dsl/term/wildcard/) | 搜索包含与通配符模式相匹配的术语的文档。
[`regexp`]({{site.url}}{{site.baseurl}}/query-dsl/term/regexp/) | 搜索包含与正则表达式匹配的术语的文档


---
layout: default
title: 复合查询
has_children: true
has_toc: false
nav_order: 40
redirect_from: 
  - /query-dsl/compound/index/
  - /query-dsl/query-dsl/compound/
---

# 复合查询

复合查询用作多个叶子或复合条款的包装器，以结合其结果或修改其行为。

下表列出了所有复合查询类型。

查询类型| 描述
:--- | :---
[`bool`]({{site.url}}{{site.baseurl}}/query-dsl/compound/bool/) （布尔）| 将多个查询子句与布尔逻辑相结合。
[`boosting`]({{site.url}}{{site.baseurl}}/query-dsl/compound/boosting/) | 更改文档的相关性评分而不将其从搜索结果中删除。返回匹配的文档`positive` 查询，但降低了文档在与之匹配的结果中的相关性`negative` 询问。
[`constant_score`]({{site.url}}{{site.baseurl}}/query-dsl/compound/constant-score/) | 包装查询或过滤器，并为所有匹配的文档分配恒定分数。这个分数等于`boost` 价值。
[`dis_max`]({{site.url}}{{site.baseurl}}/query-dsl/compound/disjunction-max/) （最大分离）| 返回匹配一个或多个查询条款的文档。如果文档匹配多个查询子句，则将其分配给更高的相关性分数。相关得分是使用任何匹配子句中的最高分数计算得出的，并且（可选）来自其他匹配子句的分数乘以决胜局值。
[`function_score`]({{site.url}}{{site.baseurl}}/query-dsl/compound/function-score/) | 重新计算使用您定义的函数返回查询的文档的相关性评分。
[`hybrid`]({{site.url}}{{site.baseurl}}/query-dsl/compound/hybrid/) | 将来自多个查询的相关性分数组合为给定文档的一个分数。


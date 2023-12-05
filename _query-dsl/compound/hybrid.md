---
layout: default
title: 混合
parent: 复合查询
grand_parent: 查询DSL
nav_order: 70
---

# 混合查询

您可以使用混合查询将来自多个查询的相关性分数组合为给定文档的一个分数。混合查询包含一个或多个查询的列表，并独立计算每个子查询的碎片级别的文档分数。子查询重写是在协调节点级别执行的，以避免重复计算。

## 例子

使用之前`hybrid` 查询，您必须使用一个配置搜索管道[`normalization-processor`]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/normalization-processor/) （看[这个示例]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/normalization-processor#example)）。

要尝试示例，请按照[语义搜索教程]({{site.url}}{{site.baseurl}}/ml-commons-plugin/semantic-search#tutorial)。

## 参数

下表列出了所有顶部-支持的级别参数`hybrid` 查询。

范围| 描述
：--- | ：---
`queries` | 用于匹配文档的一个或多个查询子句的数组。文档必须匹配至少一个查询条款，以便在结果中返回。所有查询子句中的文档的相关性分数都通过应用一个分为一个分数[搜索管道]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/index/)。最大查询子句为5。

## 禁用混合查询

默认情况下，启用了混合查询。要禁用集群中的混合查询，请设置`plugins.neural_search.hybrid_search_disabled` 设置为`true` 在`opensearch.yml`。


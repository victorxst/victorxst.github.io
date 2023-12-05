---
layout: default
title: 专业查询
has_children: true
nav_order: 65
has_toc: false
---

# 专业查询

OpenSearch支持以下专业查询：

- `distance_feature`：根据原点与文档的动态计算距离计算文档分数`date`，`date_nanos`， 或者`geo_point` 字段。此查询可以跳过非-竞争激烈。

- `more_like_this`：查找类似于提供的文档，文档或收集文档的文档。

- [`neural`]({{site.url}}{{site.baseurl}}/query-dsl/specialized/neural/)：用于矢量字段搜索[神经搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-search/)。

- [`neural_sparse`]({{site.url}}{{site.baseurl}}/query-dsl/specialized/neural-sparse/)：用于矢量字段搜索[稀疏的神经搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-sparse-search/)。

- `percolate`：查找与提供文档匹配的查询（存储为文档）。

- `rank_feature`：根据数字特征的值计算得分。此查询可以跳过非-竞争激烈。

- `script`：使用脚本作为过滤器。

- [`script_score`]({{site.url}}{{site.baseurl}}/query-dsl/specialized/script-score/)：使用脚本计算匹配文档的自定义分数。

- `wrapper`：接受其他查询为JSON或YAML字符串。


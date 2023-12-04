---
layout: default
title: 公制聚合
has_children: true
has_toc: false
nav_order: 2
redirect_from:
  - /opensearch/metric-agg/
  - /query-dsl/aggregations/metric-agg/
  - /aggregations/metric-agg/
  - /query-dsl/aggregations/metric/
---

# 公制聚合

公制聚合使您可以执行简单的计算，例如查找字段的最小，最大值和平均值。

## 度量聚合的类型

有两种类型的度量聚合：单个-价值度量聚合和多重-价值度量聚合。

### 单身的-价值度量聚合

单身的-值度量聚合返回一个单一度量，例如`sum`，`min`，`max`，`avg`，`cardinality`， 或者`value_count`。

### 多-价值度量聚合

多-值度量聚合返回多个度量。这些包括`stats`，`extended_stats`，`matrix_stats`，`percentile`，`percentile_ranks`，`geo_bound`，`top_hits`， 和`scripted_metric`。

## 支持的度量聚合

OpenSearch支持以下指标聚合：

- [平均的]({{site.url}}{{site.baseurl}}/aggregations/metric/average/)
- [基数]({{site.url}}{{site.baseurl}}/aggregations/metric/cardinality/)
- [扩展统计数据]({{site.url}}{{site.baseurl}}/aggregations/metric/extended-stats/)
- [Geobounds]({{site.url}}{{site.baseurl}}/aggregations/metric/geobounds/)
- [矩阵统计]({{site.url}}{{site.baseurl}}/aggregations/metric/matrix-stats/)
- [最大限度]({{site.url}}{{site.baseurl}}/aggregations/metric/maximum/)
- [最低限度]({{site.url}}{{site.baseurl}}/aggregations/metric/minimum/)
- [百分位数]({{site.url}}{{site.baseurl}}/aggregations/metric/percentile-ranks/)
- [百分位数]({{site.url}}{{site.baseurl}}/aggregations/metric/percentile/)
- [脚本公制]({{site.url}}{{site.baseurl}}/aggregations/metric/scripted-metric/)
- [统计]({{site.url}}{{site.baseurl}}/aggregations/metric/stats/)
- [和]({{site.url}}{{site.baseurl}}/aggregations/metric/sum/)
- [热门单曲]({{site.url}}{{site.baseurl}}/aggregations/metric/top-hits/)
- [value count]({{stite.url}}) {{site.baseurl}}/gentregations/metric/value-数数/[价值计数]({{site.url}}{{site.baseurl}}/aggregations/metric/value-count/)[value count]({{stite.url}}) {{site.baseurl}}/gentregations/metric/value-数数/


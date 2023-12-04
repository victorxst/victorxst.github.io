---
layout: default
title: 桶聚合
has_children: true
has_toc: false
nav_order: 3
redirect_from:
  - /opensearch/bucket-agg/
  - /query-dsl/aggregations/bucket-agg/
  - /query-dsl/aggregations/bucket/
  - /aggregations/bucket-agg/
---

# 水桶聚集

存储桶汇总将一组文档分类为存储桶。存储桶聚合的类型确定给定文档的存储桶。

您可以使用Bucket聚合来实现刻面导航（通常作为侧边栏放置在搜索结果登录页面上），以帮助您的用户过滤结果。

## 支持的铲斗聚合

OpenSearch支持以下水桶聚合：

- [邻接矩阵]({{site.url}}{{site.baseurl}}/aggregations/bucket/adjacency-matrix/)
- [日期直方图]({{site.url}}{{site.baseurl}}/aggregations/bucket/date-histogram/)
- [日期范围]({{site.url}}{{site.baseurl}}/aggregations/bucket/date-range/)
- [多样化的采样器]({{site.url}}{{site.baseurl}}/aggregations/bucket/diversified-sampler/)
- [筛选]({{site.url}}{{site.baseurl}}/aggregations/bucket/filter/)
- [过滤器]({{site.url}}{{site.baseurl}}/aggregations/bucket/filters/)
- [大地抗衡]({{site.url}}{{site.baseurl}}/aggregations/bucket/geo-distance/)
- [Geohash Grid]({{site.url}}{{site.baseurl}}/aggregations/bucket/geohash-grid/)
- [Geohex网格]({{site.url}}{{site.baseurl}}/aggregations/bucket/geohex-grid/)
- [Geotile Grid]({{site.url}}{{site.baseurl}}/aggregations/bucket/geotile-grid/)
- [全球的]({{site.url}}{{site.baseurl}}/aggregations/bucket/global/)
- [直方图]({{site.url}}{{site.baseurl}}/aggregations/bucket/histogram/)
- [IP范围]({{site.url}}{{site.baseurl}}/aggregations/bucket/ip-range/)
- [丢失的]({{site.url}}{{site.baseurl}}/aggregations/bucket/missing/)
- [并发-术语]({{site.url}}{{site.baseurl}}/aggregations/bucket/multi-terms/)
- [嵌套]({{site.url}}{{site.baseurl}}/aggregations/bucket/nested/)
- [范围]({{site.url}}{{site.baseurl}}/aggregations/bucket/range/)
- [反向嵌套]({{site.url}}{{site.baseurl}}/aggregations/bucket/reverse-nested/)
- [采样器]({{site.url}}{{site.baseurl}}/aggregations/bucket/sampler/)
- [重要术语]({{site.url}}{{site.baseurl}}/aggregations/bucket/significant-terms/)
- [重要的文字]({{site.url}}{{site.baseurl}}/aggregations/bucket/significant-text/)
- [enter]({{stite.url}}) {{site.baseurl}}/gentregations/bucket/tenter/enter/[术语]({{site.url}}{{site.baseurl}}/aggregations/bucket/terms/)[enter]({{stite.url}}) {{site.baseurl}}/gentregations/bucket/tenter/enter/


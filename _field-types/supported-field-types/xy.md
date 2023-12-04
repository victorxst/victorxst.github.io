---
layout: default
title: Cartesian字段类型
nav_order: 57
has_children: true
has_toc: false
grand_parent: 支持的字段类型
redirect_from:
  - /opensearch/supported-field-types/xy/
  - /field-types/xy/
---

# 笛卡尔现场类型

笛卡尔田地类型有助于索引和搜索两者中的点和形状-维笛卡尔坐标系。笛卡尔现场类型与[地理]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geographic/) 现场类型，除了它们代表笛卡尔平面上的点和形状，这不是基于地球的-固定地面参考系统。在平面上计算距离要比在球体上计算距离更有效，因此对于笛卡尔场类型而言，距离排序更快。

笛卡尔现场类型适用于虚拟现实，计算机等空间应用程序-辅助设计（CAD），游乐园和体育场地映射。

笛卡尔字段类型的坐标是单一的-精密浮动-点值。有关浮动范围和精度的信息-点值，请参阅[数字字段类型]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/numeric/)。

下表列出了OpenSearch支持的所有笛卡尔字段类型。

字段数据类型| 描述
：--- | ：---  
[`xy_point`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/xy-point/) | 一个点-尺寸笛卡尔坐标系，由X和Y坐标指定。
[`xy_shape`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/xy-shape/) | 一个形状，例如多边形或xy点的集合，-维笛卡尔坐标系。

当前，OpenSearch支持索引和搜索笛卡尔字段类型，而不是笛卡尔字段类型的汇总。如果您想查看实现的聚合，请打开一个[Github问题](https://github.com/opensearch-project/geospatial)。
{: .note}


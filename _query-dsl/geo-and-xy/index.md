---
layout: default
title: 地理和XY查询
has_children: true
nav_order: 50
redirect_from:
   - /opensearch/query-dsl/geo-and-xy/index/
   - /query-dsl/query-dsl/geo-and-xy/
   - /query-dsl/query-dsl/geo-and-xy/index/
---

# 地理和XY查询

地理和XY查询可让您搜索在地图或坐标平面上包含点和形状的字段。地理查询在地理空间数据上工作，而XY查询在两个方面工作-维数数据。在所有地理查询中，Geoshape查询与XY查询非常相似，但前者搜索[地理领域]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geographic)，而后者进行搜索[笛卡尔田]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/xy)。

## xy查询

[xy查询]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/geo-and-xy/xy) 搜索在笛卡尔坐标系中包含几何形状的文档。这些几何形状可以在[`xy_point`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/xy-point) 田野，支持点，以及[`xy_shape`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/xy-shape) 支持点，线，圆和多边形的字段。

xy查询返回包含的文档：
- XY形状和XY点具有与所提供形状的四个空间关系之一：`INTERSECTS`，`DISJOINT`，`WITHIN`， 或者`CONTAINS`。
- 与提供形状相交的XY点。

## 地理查询

地理查询搜索包含地理空间几何形状的文档。这些几何形状可以在[`geo_point`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geo-point) 字段，在地图上支持点，并且[`geo_shape`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geo-shape) 支持点，线，圆和多边形的字段。

OpenSearch提供以下地理查询类型：

- [**地理-边界框查询**]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/geo-and-xy/geo-bounding-box/)：带有边界框中的地理点字段值的返回文档。
- **GeoDistance查询** 返回文档，带有指定距离的地理点距离的地理点。
- **Geopoygon查询** 带有多边形内的地理点的返回文档。
- **Geoshape查询** 包含的返回文档：
    - 与所提供形状具有四个空间关系之一的地理和地理点：`INTERSECTS`，`DISJOINT`，`WITHIN`， 或者`CONTAINS`。
    - 与提供形状相交的地理点


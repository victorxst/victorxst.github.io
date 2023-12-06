---
layout: default
title: Geoshape
nav_order: 57
has_children: false
parent: 地理字段类型
grand_parent: 支持的字段类型
redirect_from:
  - /opensearch/supported-field-types/geo-shape/
  - /field-types/geo-shape/
---

# Geoshape字段类型

Geoshape场类型包含地理形状，例如多边形或地理点的集合。要为GeoShape索引，请将搜索切换到三角形网格中，然后将每个三角形存储在BKD树上。这提供了10 <sup>-7 </sup>十进制精度，代表附近-完美的空间分辨率。该过程的性能主要受您索引多边形的顶点的数量影响。

## 例子

使用GeoShape字段类型创建映射：

```json
PUT testindex
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_shape"
      }
    }
  }
}
```
{% include copy-curl.html %}

## 格式

geoshapes可以以以下格式进行索引：

- [Geojson](https://geojson.org/)
- [出色地-已知文本（WKT）](https://docs.opengeospatial.org/is/12-063r5/12-063r5.html)

在Geojson和WKT中，必须在`longitude, latitude` 在坐标数组中订购。请注意，经度首先以这种格式出现。
{: .note}

## Geoshape类型

下表描述了可能的Geoshape类型及其与Geojson和WKT类型的关系。

OpenSearch类型| Geojson类型| WKT类型| 描述
:--- | :--- | :---  | ：--- 
[`point`](#point) | 观点| 观点| 由纬度和经度指定的地理点。OpenSearch使用世界测量系统（WGS84）坐标。
[`linestring`](#linestring) | linestring| linestring| 由两个或多个点指定的线。可能是连接线段的直线或路径。
[`polygon`](#polygon) | 多边形| 多边形| 由坐标形式的顶点列表指定的多边形。多边形必须关闭，这意味着最后一点必须与第一个点相同。因此，创建一个n-GON，N+1个顶点。最小顶点数为四个，这会产生一个三角形。
[`multipoint`](#multipoint) | 多点| 多点| 一系列未连接的离散相关点。
[`multilinestring`](#multilinestring) | 多势| 多势| 一系列的衬里。
[`multipolygon`](#multipolygon) | 多重子| 多重子| 多边形阵列。
[`geometrycollection`](#geometry-collection) | 几何收获| 几何收获| 可能是不同类型的Geoshapes集合。
[`envelope`](#envelope) | N/A。| bbox| 由鞋面指定的边界矩形-左右-右顶点。

## 观点

一个点是一对由纬度和经度指定的坐标。

索引格式的观点：

```json
PUT testindex/_doc/1
{
  "location" : {
    "type" : "point",
    "coordinates" : [74.00, 40.71]
  }
}
```
{% include copy-curl.html %}

索引wkt格式的点：

```json
PUT testindex/_doc/1
{
  "location" : "POINT (74.0060 40.7128)"
}
```
{% include copy-curl.html %}

## linestring

linestring是由两个或多个点指定的线。如果点是共线，则衬里是一条直线。否则，linestring表示线段制成的路径。

索引geojson格式的统一性：

```json
PUT testindex/_doc/2
{
  "location" : {
    "type" : "linestring",
    "coordinates" : [[74.0060, 40.7128], [71.0589, 42.3601]]
  }
}
```
{% include copy-curl.html %}

索引wkt格式的统一性：

```json
PUT testindex/_doc/2
{
  "location" : "LINESTRING (74.0060 40.7128, 71.0589 42.3601)"
}
```
{% include copy-curl.html %}

## 多边形

多边形由以坐标形式的顶点列表指定。多边形必须关闭，这意味着最后一点必须与第一个点相同。在下面的示例中，使用四个点创建一个三角形。

Geojson要求您逆时针列出多边形的顶点。WKT不会在顶点强加特定的顺序。
{: .note}

索引以geojson格式的多边形（三角形）：

```json
PUT testindex/_doc/3
{
  "location" : {
    "type" : "polygon",
    "coordinates" : [
      [[74.0060, 40.7128], 
      [71.0589, 42.3601], 
      [73.7562, 42.6526], 
      [74.0060, 40.7128]]
    ]
  }
}
```
{% include copy-curl.html %}

索引以WKT格式的多边形（三角形）：

```json
PUT testindex/_doc/3
{
  "location" : "POLYGON ((74.0060 40.7128, 71.0589 42.3601, 73.7562 42.6526, 74.0060 40.7128))"
}
```
{% include copy-curl.html %}

多边形可能有孔。在这种情况下，`coordinates` 字段将包含多个数组。第一个阵列代表外多边形，每个后续阵列代表一个孔。孔表示为多边形，并指定为坐标阵列。

Geojson要求您逆时针列出多边形的顶点和孔的顶点。WKT不会在顶点强加特定的顺序。
{: .note}

索引一个多边形（三角形），带有三角形孔，格式：

```json
PUT testindex/_doc/4
{
  "location" : {
    "type" : "polygon",
    "coordinates" : [
      [[74.0060, 40.7128], 
      [71.0589, 42.3601], 
      [73.7562, 42.6526], 
      [74.0060, 40.7128]],
      
      [[72.6734,41.7658], 
      [72.6506, 41.5623], 
      [73.0515, 41.5582], 
      [72.6734, 41.7658]]
    ]
  }
}
```
{% include copy-curl.html %}

索引A多边形（三角形），具有WKT格式的三角形孔：

```json
PUT testindex/_doc/4
{
  "location" : "POLYGON ((40.7128 74.0060, 42.3601 71.0589, 42.6526 73.7562, 40.7128 74.0060), (41.7658 72.6734, 41.5623 72.6506, 41.5582 73.0515, 41.7658 72.6734))"
}
```
{% include copy-curl.html %}

在OpenSearch中，您可以通过顺时针或逆时针列出其顶点来指定多边形。这对于不越过日期线的多边形（比180＆deg窄;）很好。但是，越过日期线的多边形（大于180＆deg;）可能是模棱两可的，因为WKT不会在顶点上施加特定的顺序。因此，您必须通过逆时针列出其顶点来指定越过日期线的多边形。

您可以定义一个[`orientation`](#parameters) 参数在映射时间指定顶点遍历顺序：

```json
PUT testindex
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_shape",
        "orientation" : "left"
      }
    }
  }
}
```
{% include copy-curl.html %}

随后，索引文件可以覆盖`orientation` 环境：

```json
PUT testindex/_doc/3
{
  "location" : {
    "type" : "polygon",
    "orientation" : "cw",
    "coordinates" : [
      [[74.0060, 40.7128], 
      [71.0589, 42.3601], 
      [73.7562, 42.6526], 
      [74.0060, 40.7128]]
    ]
  }
}
```
{% include copy-curl.html %}

## 多点

多点是未连接的离散相关点的数组。

索引geojson格式的多点：

```json
PUT testindex/_doc/6
{
  "location" : {
    "type" : "multipoint",
    "coordinates" : [
      [74.0060, 40.7128], 
      [71.0589, 42.3601]
    ]
  }
}
```
{% include copy-curl.html %}

索引WKT格式的多点：

```json
PUT testindex/_doc/6
{
  "location" : "MULTIPOINT (74.0060 40.7128, 71.0589 42.3601)"
}
```
{% include copy-curl.html %}

## 多势

多势是一系列的固定线。

索引geojson格式的统一性：

```json
PUT testindex/_doc/2
{
  "location" : {
    "type" : "multilinestring",
    "coordinates" : [
      [[74.0060, 40.7128], [71.0589, 42.3601]],
      [[73.7562, 42.6526], [72.6734, 41.7658]]
      ]
  }
}
```
{% include copy-curl.html %}

索引wkt格式的统一性：

```json
PUT testindex/_doc/2
{
  "location" : "MULTILINESTRING ((74.0060 40.7128, 71.0589 42.3601), (73.7562 42.6526, 72.6734 41.7658))"
}
```
{% include copy-curl.html %}

## 多重子

多角形是多边形的阵列。在此示例中，第一个多边形包含一个孔，而第二个则没有。

索引以geojson格式的多重分子：

```json
PUT testindex/_doc/4
{
  "location" : {
    "type" : "multipolygon",
    "coordinates" : [
    [
      [[74.0060, 40.7128], 
      [71.0589, 42.3601], 
      [73.7562, 42.6526], 
      [74.0060, 40.7128]],
      
      [[72.6734,41.7658], 
      [72.6506, 41.5623], 
      [73.0515, 41.5582], 
      [72.6734, 41.7658]]
    ],
    [
      [[73.9776, 40.7614], 
      [73.9554, 40.7827], 
      [73.9631, 40.7812], 
      [73.9776, 40.7614]]
      ]
    ]
  }
}
```
{% include copy-curl.html %}

索引以WKT格式的多重分子：

```json
PUT testindex/_doc/4
{
  "location" : "MULTIPOLYGON (((40.7128 74.0060, 42.3601 71.0589, 42.6526 73.7562, 40.7128 74.0060), (41.7658 72.6734, 41.5623 72.6506, 41.5582 73.0515, 41.7658 72.6734)), ((73.9776 40.7614, 73.9554 40.7827, 73.9631 40.7812, 73.9776 40.7614)))"
}
```
{% include copy-curl.html %}

## 几何收集

几何集合是可能具有不同类型的GeoShapes的集合。

索引以Geojson格式的几何收藏：

```json
PUT testindex/_doc/7
{
  "location" : {
    "type": "geometrycollection",
    "geometries": [
      {
        "type": "point",
        "coordinates": [74.0060, 40.7128]
      },
      {
        "type": "linestring",
        "coordinates": [[73.7562, 42.6526], [72.6734, 41.7658]]
      }
    ]
  }
}
```
{% include copy-curl.html %}

索引以WKT格式的几何收集：

```json
PUT testindex/_doc/7
{
  "location" : "GEOMETRYCOLLECTION (POINT (74.0060 40.7128), LINESTRING(73.7562 42.6526, 72.6734 41.7658))"
}
```
{% include copy-curl.html %}

## 信封

信封是一个由鞋面指定的边界矩形-左右-右顶点。Geojson格式是`[[minLon, maxLat], [maxLon, minLat]]`。

索引格式的信封：

```json
PUT testindex/_doc/2
{
  "location" : {
    "type" : "envelope",
    "coordinates" : [[71.0589, 42.3601], [74.0060, 40.7128]]
  }
}
```
{% include copy-curl.html %}

以WKT格式，使用`BBOX (minLon, maxLon, maxLat, minLat)`。

WKT Bbox格式的索引一个信封：

```json
PUT testindex/_doc/8
{
  "location" : "BBOX (71.0589, 74.0060, 42.3601, 40.7128)"
}
```
{% include copy-curl.html %}

## 参数

下表列出了GeoShape字段类型接受的参数。所有参数都是可选的。

范围| 描述
:--- | :--- 
`coerce` | 布尔值指定是否会自动关闭未关闭未关闭的线性环。默认为`false`。
`ignore_malformed` | 一个布尔值指定的是忽略畸形的Geojson或WKT Geoshapes，而不是抛出例外。默认为`false` （当Geoshapes畸形时，请例外）。
`ignore_z_value` | 特定于具有三个坐标的点。如果`ignore_z_value` 是`true`，第三个坐标没有索引，但仍存储在_source字段中。如果`ignore_z_value` 是`false`，一个例外。默认为`true`。
`orientation` | 指定Geoshape坐标列表中顶点的遍历顺序。`orientation` 采用以下值：<br> 1.右：逆时针。通过使用以下字符串之一（大写或小写）来指定正确的方向：`right`，`counterclockwise`，`ccw`。<br> 2.左：顺时针。使用以下一个字符串之一（大写或小写）来指定左定向：`left`，`clockwise`，`cw`。单个文档可以覆盖此值。<br>默认值为`RIGHT`。


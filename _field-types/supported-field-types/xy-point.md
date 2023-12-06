---
layout: default
title: xy点
nav_order: 58
has_children: false
parent: Cartesian字段类型
grand_grand_parent: 支持的字段类型
redirect_from:
  - /opensearch/supported-field-types/xy-point/
  - /field-types/xy-point/
---

# XY点字段类型

xy点字段类型包含两个点-尺寸笛卡尔坐标系，由X和Y坐标指定。它是基于Lucene的[xypoint](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/geo/XYPoint.html) 字段类型。XY点字段类型类似于[地理点]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geo-point/) 现场类型，但没有地理点的范围限制。XY点的坐标是单个的-精密浮动-点值。有关浮动范围和精度的信息-点值，请参阅[数字字段类型]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/numeric/)。

## 例子

使用XY点字段类型创建映射：

```json
PUT testindex1
{
  "mappings": {
    "properties": {
      "point": {
        "type": "xy_point"
      }
    }
  }
}
```
{% include copy-curl.html %}

## 格式

XY点可以以以下格式索引：

- 具有X和Y坐标的对象

```json
PUT testindex1/_doc/1
{
  "point": { 
    "x": 0.5,
    "y": 4.5
  }
}
```
{% include copy-curl.html %}

- 一条字符串"`x`, `y`" 格式

```json
PUT testindex1/_doc/2
{
  "point": "0.5, 4.5" 
}
```
{% include copy-curl.html %}

- [`x`，`y`] 格式

```json
PUT testindex1/_doc/3
{
  "point": [0.5, 4.5] 
}
```
{% include copy-curl.html %}

- A[出色地-已知文本（WKT）](https://docs.opengeospatial.org/is/12-063r5/12-063r5.html) 指向"POINT(`x` `y`)" 格式

```json
PUT testindex1/_doc/4
{
  "point": "POINT (0.5 4.5)"
}
```
{% include copy-curl.html %}

- Geojson格式

```json
PUT testindex1/_doc/5
{
  "point" : {
    "type" : "Point",
    "coordinates" : [0.5, 4.5]        
  }
}
```
{% include copy-curl.html %}

在所有XY点格式中，必须在`x, y` 命令。
{: .note}}

## 参数

下表列出了XY点字段类型接受的参数。所有参数都是可选的。

范围| 描述
:--- | :--- 
`ignore_malformed` | 布尔值指定忽略畸形值而不引发异常的值。默认为`false`。
`ignore_z_value` | 特定于具有三个坐标的点。如果`ignore_z_value` 是`true`，第三个坐标没有索引，但仍存储在_source字段中。如果`ignore_z_value` 是`false`，一个例外。
[`null_value`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/index#null-value) | 用于代替的值`null`。该值必须与字段相同。如果未指定此参数，则该字段在其值为时被视为丢失`null`。默认为`null`


---
layout: default
title: 地理点
nav_order: 56
has_children: false
parent: 地理字段类型
grand_parent: 支持的字段类型
redirect_from:
  - /opensearch/supported-field-types/geo-point/
  - /field-types/geo-point/
---

# 地理点字段类型

地理点字段类型包含由纬度和经度指定的地理点。

## 例子

使用地理点字段类型创建映射：

```json
PUT testindex1
{
  "mappings": {
    "properties": {
      "point": {
        "type": "geo_point"
      }
    }
  }
}
```
{% include copy-curl.html %}

## 格式

地理点可以以以下格式索引：

- 具有纬度和经度的对象

```json
PUT testindex1/_doc/1
{
  "point": { 
    "lat": 40.71,
    "lon": 74.00
  }
}
```
{% include copy-curl.html %}

- 一条字符串"`latitude`,`longitude`" 格式

```json
PUT testindex1/_doc/2
{
  "point": "40.71,74.00" 
}
```
{% include copy-curl.html %}

- 地理船

```json
PUT testindex1/_doc/3
{
  "point": "txhxegj0uyp3"
}
```
{% include copy-curl.html %}

- [`longitude`，`latitude`] 格式

```json
PUT testindex1/_doc/4
{
  "point": [74.00, 40.71] 
}
```
{% include copy-curl.html %}

- A[出色地-已知文字](https://docs.opengeospatial.org/is/12-063r5/12-063r5.html) 指向"POINT(`longitude` `latitude`)" 格式

```json
PUT testindex1/_doc/5
{
  "point": "POINT (74.00 40.71)"
}
```
{% include copy-curl.html %}

- Geojson格式，`coordinates` 在[`longitude`，`latitude`] 格式

```json
PUT testindex1/_doc/6
{
  "point": {
    "type": "Point",
    "coordinates": [74.00, 40.71]
  }
}
```
{% include copy-curl.html %}

## 参数

下表列出了通过地理点字段类型接受的参数。所有参数都是可选的。

范围| 描述
:--- | :--- 
`ignore_malformed` | 布尔值指定忽略畸形值而不引发异常的值。纬度的有效值为[-90，90]。经度的有效值是[-180，180]。默认为`false`。
`ignore_z_value` | 特定于具有三个坐标的点。如果`ignore_z_value` 是`true`，第三个坐标没有索引，但仍存储在_source字段中。如果`ignore_z_value` 是`false`，一个例外。
[`null_value`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/index#null-value) | 用于代替的值`null`。必须与字段相同。如果未指定此参数，则该字段在其值为时被视为丢失`null`。默认为`null`


---
layout: default
title: Geodistance
parent: 桶聚合
grand_parent: 聚合
nav_order: 70
redirect_from:
  - /query-dsl/aggregations/bucket/geo-distance/
---

# 地球固定聚集

这`geo_distance` 基于距离距离的距离，聚集组将文档记录到同心圆`geo_point` 场地。
与`range` 聚集，除了它在地理位置上工作。

例如，您可以使用`geo_distance` 聚集以在您1公里以内找到所有披萨。搜索结果仅限于您指定的1 km半径，但是您可以在2公里内添加另一个结果。

您只能使用`geo_distance` 在映射的字段上的聚合`geo_point`。

一个点是单个地理坐标，例如您的智能显示的当前位置-电话。OpenSearch中的一个点表示如下：

```json
{
  "location": {
    "type": "point",
    "coordinates": {
      "lat": 83.76,
      "lon": -81.2
    }
  }
}
```

您也可以将纬度和经度指定为数组`[-81.20, 83.76]` 或作为字符串`"83.76, -81.20"`

该表列出了一个相关字段`geo_distance` 聚合：

场地| 描述| 必需的
：--- | ：--- |：---
`field` |  指定要处理的地理点字段。| 是的
`origin` |  指定用于计算距离的地理点。| 是的
`ranges`  |  根据其距离目标点的距离指定范围列表来收集文档。| 是的
`unit` |  定义在`ranges` 大批。这`unit` 默认为`m` （米），但是您可以切换到其他单位`km` （公里），`mi` （英里），，`in` （英寸），`yd` （码），`cm` （厘米），以及`mm` （毫米）。| 不
`distance_type` | 指定OpenSearch如何计算距离。默认值为`sloppy_arc` （更快但准确），但也可以设置为`arc` （较慢但最准确）或`plane` （最快但最不准确）。由于较高的错误边距，请使用`plane` 仅适用于小地理区域。| 不

语法如下：

```json
{
  "aggs": {
    "aggregation_name": {
      "geo_distance": {
        "field": "field_1",
        "origin": "x, y",
        "ranges": [
          {
            "to": "value_1"
          },
          {
            "from": "value_2",
            "to": "value_3"
          },
          {
            "from": "value_4"
          }
        ]
      }
    }
  }
}
```

此示例从以下距离从一个`geo-point` 场地：

- 不到10公里
- 从10到20公里
- 从20到50公里
- 从50到100公里
- 超过100公里

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "position": {
      "geo_distance": {
        "field": "geo.coordinates",
        "origin": {
          "lat": 83.76,
          "lon": -81.2
        },
        "ranges": [
          {
            "to": 10
          },
          {
            "from": 10,
            "to": 20
          },
          {
            "from": 20,
            "to": 50
          },
          {
            "from": 50,
            "to": 100
          },
          {
            "from": 100
          }
        ]
      }
    }
  }
}
```
{% include copy-curl.html %}

#### 示例响应

```json
...
"aggregations" : {
  "position" : {
    "buckets" : [
      {
        "key" : "*-10.0",
        "from" : 0.0,
        "to" : 10.0,
        "doc_count" : 0
      },
      {
        "key" : "10.0-20.0",
        "from" : 10.0,
        "to" : 20.0,
        "doc_count" : 0
      },
      {
        "key" : "20.0-50.0",
        "from" : 20.0,
        "to" : 50.0,
        "doc_count" : 0
      },
      {
        "key" : "50.0-100.0",
        "from" : 50.0,
        "to" : 100.0,
        "doc_count" : 0
      },
      {
        "key" : "100.0-*",
        "from" : 100.0,
        "doc_count" : 14074
      }
    ]
  }
 }
}
```

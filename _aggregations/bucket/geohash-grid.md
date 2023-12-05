---
layout: default
title: Geohash grid
parent: 桶聚合
grand_parent: 聚合
nav_order: 80
redirect_from:
  - /query-dsl/aggregations/bucket/geohash-grid/
---

# GeoHash网格聚集

这`geohash_grid` 聚合存储桶文档用于地理分析。它将一个地理区域组织成一个不同大小或精确度的较小区域的网格。较低的精度值代表较大的地理区域，较高的值代表较小，更精确的地理区域。您可以汇总文档[地理点]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geo-point/) 或者[Geoshape]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geo-shape/) 使用GeoHash网格聚集的字段。一个值得注意的区别是，地理点仅存在于一个桶中，但是Geoshape在与其相交的所有GeoHash网格单元中计数。

查询返回的结果数可能太多了，无法在地图上单独显示每个地理点。这`geohash_grid` 通过在定义的精度（1至12之间；默认值为5）的精度级别，通过计算每个点的Geohash来合计的聚合存储桶一起计算地理点。要了解有关GeoHash的更多信息，请参阅[维基百科](https://en.wikipedia.org/wiki/Geohash)。

Web日志示例数据分布在大型地理区域上，因此您可以使用较低的精度值。您可以通过增加精度值来放大此地图：

```json
GET opensearch_dashboards_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "geo_hash": {
      "geohash_grid": {
        "field": "geo.coordinates",
        "precision": 4
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
  "geo_hash" : {
    "buckets" : [
      {
        "key" : "c1cg",
        "doc_count" : 104
      },
      {
        "key" : "dr5r",
        "doc_count" : 26
      },
      {
        "key" : "9q5b",
        "doc_count" : 20
      },
      {
        "key" : "c20g",
        "doc_count" : 19
      },
      {
        "key" : "dr70",
        "doc_count" : 18
      }
      ...
    ]
  }
 }
}
```

您可以使用OpenSearch仪表板在地图上可视化汇总响应。

您希望汇总的准确性越准确，由于汇总必须计算的存储桶数量，因此OpenSearch消耗的资源越多。默认情况下，OpenSearch不会生成超过10,000个存储桶。您可以使用`size` 属性，但请记住，该性能可能会在包括数千个水桶组成的非常广泛的查询中受到影响。

## 聚集的Geoshapes

要在Geoshape字段上运行聚合，请首先创建索引并映射`location` 字段作为`geo_shape`：

```json
PUT national_parks
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

接下来，将一些文档索引到`national_parks` 指数：

```json
PUT national_parks/_doc/1
{
  "name": "Yellowstone National Park",
  "location":
  {"type": "envelope","coordinates": [ [-111.15, 45.12], [-109.83, 44.12] ]}
}
```
{% include copy-curl.html %}

```json
PUT national_parks/_doc/2
{
  "name": "Yosemite National Park",
  "location": 
  {"type": "envelope","coordinates": [ [-120.23, 38.16], [-119.05, 37.45] ]}
}
```
{% include copy-curl.html %}

```json
PUT national_parks/_doc/3
{
  "name": "Death Valley National Park",
  "location": 
  {"type": "envelope","coordinates": [ [-117.34, 37.01], [-116.38, 36.25] ]}
}
```
{% include copy-curl.html %}

您可以在`location` 字段如下：

```json
GET national_parks/_search
{
  "aggregations": {
    "grouped": {
      "geohash_grid": {
        "field": "location",
        "precision": 1
      }
    }
  }
}
```
{% include copy-curl.html %}

当汇总Geoshapes时，可以对多个存储桶进行计数一个Geoshape，因为它与多个网格单元重叠：

<details open markdown="block">
  <summary>
    回复
  </summary>
  {： 。文本-三角洲}
  
```json
{
  "took" : 24,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "national_parks",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "Yellowstone National Park",
          "location" : {
            "type" : "envelope",
            "coordinates" : [
              [
                -111.15,
                45.12
              ],
              [
                -109.83,
                44.12
              ]
            ]
          }
        }
      },
      {
        "_index" : "national_parks",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "name" : "Yosemite National Park",
          "location" : {
            "type" : "envelope",
            "coordinates" : [
              [
                -120.23,
                38.16
              ],
              [
                -119.05,
                37.45
              ]
            ]
          }
        }
      },
      {
        "_index" : "national_parks",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "name" : "Death Valley National Park",
          "location" : {
            "type" : "envelope",
            "coordinates" : [
              [
                -117.34,
                37.01
              ],
              [
                -116.38,
                36.25
              ]
            ]
          }
        }
      }
    ]
  },
  "aggregations" : {
    "grouped" : {
      "buckets" : [
        {
          "key" : "9",
          "doc_count" : 3
        },
        {
          "key" : "c",
          "doc_count" : 1
        }
      ]
    }
  }
}
```
</details>

当前，OpenSearch通过API支持GeoShape聚合，但在OpenSearch仪表板可视化中不支持GEOSHAPE聚合。如果您想查看为可视化实施的GeoShape聚合，请访问相关的[Github问题](https://github.com/opensearch-project/dashboards-maps/issues/250)。
{: .note}

## 支持的参数

GeoHash网格聚合请求支持以下参数。

范围| 数据类型| 描述
:--- | :--- | :--- 
场地| 细绳| 执行聚合的字段。该字段必须映射为`geo_point` 或者`geo_shape` 场地。如果字段包含一个数组，则所有数组值均已汇总。必需的。
精确| 整数| 缩放水平用于确定网格单元以进行铲斗结果。有效值在[0，15]范围内。选修的。默认值为5。
边界| 目的| 用于过滤地理点和geoshapes的边界框。边界框由鞋面定义-左右-右顶点。只有与此边界框相交或完全由该边界框包含的形状包含在聚合输出中。这些顶点在以下格式之一中指定为地理点：<br>- 具有纬度和经度的对象<br>- 阵列[`longitude`，`latitude`](format<br>- A string in the "`latitude`,`longitude`" format<br>- A geohash <br>- WKT<br> See the [geopoint formats]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geo-point#formats) 用于格式化示例。选修的。
尺寸| 整数| 返回的最大存储桶数。当水桶比`size`，OpenSearch返回带有更多文档的存储桶。选修的。默认值为10,000。
shard_size| 整数| 从每个碎片返回的最大存储桶数。选修的。默认值为最大（10，`size` ＆middot;碎片数），这提供了更准确的更优先级存储桶的计数


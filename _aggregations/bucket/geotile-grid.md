---
layout: default
title: Geotile grid
parent: 桶聚合
grand_parent: 聚合
nav_order: 87
redirect_from:
  - /query-dsl/aggregations/bucket/geotile-grid/
---

# Geotile网格聚集

Geotile网格聚集组记录到网格细胞中进行地理分析。每个网格单元对应于[地图瓷砖](https://en.wikipedia.org/wiki/Tiled_web_map) 并使用`{zoom}/{x}/{y}` 格式。您可以汇总文档[地理点]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geo-point/) 或者[Geoshape]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geo-shape/) 使用Geotile网格聚集的字段。一个值得注意的区别是，地理点仅存在于一个桶中，但是在与其相交的所有Geoshape中，GeoShape都计数。

## 精确

这`precision` 参数控制确定网格单元大小的粒度水平。精度越低，网格单元越大。

以下示例说明了低-精度和高-精确汇总请求。

首先，创建索引并映射`location` 字段作为`geo_point`：

```json
PUT national_parks
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      }
    }
  }
}
```
{% include copy-curl.html %}

将以下文档索引到样本索引：

```json
PUT national_parks/_doc/1
{
  "name": "Yellowstone National Park",
  "location": "44.42, -110.59" 
}
```
{% include copy-curl.html %}

```json
PUT national_parks/_doc/2
{
  "name": "Yosemite National Park",
  "location": "37.87, -119.53" 
}
```
{% include copy-curl.html %}

```json
PUT national_parks/_doc/3
{
  "name": "Death Valley National Park",
  "location": "36.53, -116.93" 
}
```
{% include copy-curl.html %}

您可以以多种格式索引地理点。有关所有支持格式的列表，请参见[地理点文档]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geo-point#formats)。
{:.note}

## 低的-精度请求

低点-精确要求将所有三个文件都结合在一起：

```json
GET national_parks/_search
{
  "aggregations": {
    "grouped": {
      "geotile_grid": {
        "field": "location",
        "precision": 1
      }
    }
  }
}
```
{% include copy-curl.html %}

您可以使用`GET` 或者`POST` HTTP方法用于Geotile网格聚合查询。
{:.note}

响应组将所有文档一起在一起，因为它们足够近以在一个网格单元格中存放：

<details open markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}

```json
{
  "took": 51,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "national_parks",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "Yellowstone National Park",
          "location": "44.42, -110.59"
        }
      },
      {
        "_index": "national_parks",
        "_id": "2",
        "_score": 1,
        "_source": {
          "name": "Yosemite National Park",
          "location": "37.87, -119.53"
        }
      },
      {
        "_index": "national_parks",
        "_id": "3",
        "_score": 1,
        "_source": {
          "name": "Death Valley National Park",
          "location": "36.53, -116.93"
        }
      }
    ]
  },
  "aggregations": {
    "grouped": {
      "buckets": [
        {
          "key": "1/0/0",
          "doc_count": 3
        }
      ]
    }
  }
}
```
</details>

## 高的-精度请求

现在跑高-精确请求：

```json
GET national_parks/_search
{
  "aggregations": {
    "grouped": {
      "geotile_grid": {
        "field": "location",
        "precision": 6
      }
    }
  }
}
```
{% include copy-curl.html %}

由于粒度较高，这三个文档都是单独贴上的：

<details open markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}
  
```json
{
  "took": 15,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "national_parks",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "Yellowstone National Park",
          "location": "44.42, -110.59"
        }
      },
      {
        "_index": "national_parks",
        "_id": "2",
        "_score": 1,
        "_source": {
          "name": "Yosemite National Park",
          "location": "37.87, -119.53"
        }
      },
      {
        "_index": "national_parks",
        "_id": "3",
        "_score": 1,
        "_source": {
          "name": "Death Valley National Park",
          "location": "36.53, -116.93"
        }
      }
    ]
  },
  "aggregations": {
    "grouped": {
      "buckets": [
        {
          "key": "6/12/23",
          "doc_count": 1
        },
        {
          "key": "6/11/25",
          "doc_count": 1
        },
        {
          "key": "6/10/24",
          "doc_count": 1
        }
      ]
    }
  }
}
```
</details>

您还可以通过在界限中提供边界信封的坐标来限制地理区域`bounds` 范围。两个都`bounds` 和`geo_bounding_box` 可以在任何一个中指定坐标[地理点格式]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geo-point#formats)。以下查询使用井-已知文本（WKT）"POINT(`longitude` `latitude`)" 格式`bounds` 范围：

```json
GET national_parks/_search
{
  "size": 0,
  "aggregations": {
    "grouped": {
      "geotile_grid": {
        "field": "location",
        "precision": 6,
        "bounds": {
            "top_left": "POINT (-120 38)",
            "bottom_right": "POINT (-116 36)"
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

响应仅包含指定范围内的两个结果：

<details open markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}
  
```json
{
  "took": 48,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "national_parks",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "Yellowstone National Park",
          "location": "44.42, -110.59"
        }
      },
      {
        "_index": "national_parks",
        "_id": "2",
        "_score": 1,
        "_source": {
          "name": "Yosemite National Park",
          "location": "37.87, -119.53"
        }
      },
      {
        "_index": "national_parks",
        "_id": "3",
        "_score": 1,
        "_source": {
          "name": "Death Valley National Park",
          "location": "36.53, -116.93"
        }
      }
    ]
  },
  "aggregations": {
    "grouped": {
      "buckets": [
        {
          "key": "6/11/25",
          "doc_count": 1
        },
        {
          "key": "6/10/24",
          "doc_count": 1
        }
      ]
    }
  }
}
```
</details>

这`bounds` 参数可以在有或没有的情况下使用`geo_bounding_box` 筛选;这两个参数是独立的，并且可以彼此之间存在任何空间关系。

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
      "geotile_grid": {
        "field": "location",
        "precision": 6
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
  {: .text-delta}

```json
{
  "took" : 3,
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
          "key" : "6/12/23",
          "doc_count" : 1
        },
        {
          "key" : "6/12/22",
          "doc_count" : 1
        },
        {
          "key" : "6/11/25",
          "doc_count" : 1
        },
        {
          "key" : "6/11/24",
          "doc_count" : 1
        },
        {
          "key" : "6/10/24",
          "doc_count" : 1
        }
      ]
    }
  }
}
```
</details>

当前，OpenSearch通过API支持GeoShape聚合，但在OpenSearch仪表板可视化中不支持GEOSHAPE聚合。如果您想查看为可视化实施的GeoShape聚合，请访问相关的[Github问题](https://github.com/opensearch-project/dashboards-maps/issues/250)。
{:.note}

## 支持的参数

Geotile网格聚合请求支持以下参数。

范围| 数据类型| 描述
:--- | :--- | :---
场地| 细绳| 包含地理点的字段。该字段必须映射为`geo_point` 场地。如果字段包含一个数组，则所有数组值均已汇总。必需的。
精确| 整数| 缩放水平用于确定网格单元以进行铲斗结果。有效值在[0，15]范围内。选修的。默认值为5。
边界| 目的| 用于过滤地理点的边界框。边界框由鞋面定义-左右-右顶点。这些顶点在以下格式之一中指定为地理点：<br>- 具有纬度和经度的对象<br>- 阵列[`longitude`，`latitude`] format<br>- A string in the "`latitude`,`longitude`" format<br>- A geohash <br>- WKT<br> See the [geopoint formats]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geo-point#formats) 用于格式化示例。选修的。
尺寸| 整数| 返回的最大存储桶数。当水桶比`size`，OpenSearch返回带有更多文档的存储桶。选修的。默认值为10,000。
shard_size| 整数| 从每个碎片返回的最大存储桶数。选修的。默认值为最大（10，`size` ＆middot;碎片数），这提供了更准确的更优先级存储桶的计数


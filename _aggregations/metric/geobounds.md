---
layout: default
title: Geobounds
parent: 公制聚合
grand_parent: 聚合
nav_order: 40
redirect_from:
  - /query-dsl/aggregations/metric/geobounds/
---

## 地理位置聚集

这`geo_bounds` 公制是多型-计算计算的值度量集合[地理边界框](https://docs.ogc.org/is/12-063r5/12-063r5.html#30) 包含给定的所有值`geo_point` 或者`geo_shape` 场地。边界框作为鞋面返回-左右-矩形在纬度和经度方面的右顶点。

以下示例返回`geo_bounds` 指标`geoip.location` 场地：

```json
GET opensearch_dashboards_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "geo": {
      "geo_bounds": {
        "field": "geoip.location"
      }
    }
  }
}
```

#### 示例响应

```json
"aggregations" : {
  "geo" : {
    "bounds" : {
      "top_left" : {
        "lat" : 52.49999997206032,
        "lon" : -118.20000001229346
      },
      "bottom_right" : {
        "lat" : 4.599999985657632,
        "lon" : 55.299999956041574
      }
    }
  }
 }
}
```

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

你可以运行`geo_bounds` 在`location` 字段如下：

```json
GET national_parks/_search
{
  "aggregations": {
    "grouped": {
      "geo_bounds": {
        "field": "location",
        "wrap_longitude": true
      }
    }
  }
}
```
{% include copy-curl.html %}

可选`wrap_longitude` 参数指定聚合返回的边界框是否可以重叠国际日期线（180＆deg; meridian）。如果`wrap_longitude` 被设定为`true`，边界框可以重叠国际日期线并返回`bounds` 较低的对象-左左距鞋面-正确的经度。的默认值`wrap_longitude` 是`true`。

响应包含地理-包围所有形状的边界框`location` 场地：

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
    "Grouped" : {
      "bounds" : {
        "top_left" : {
          "lat" : 45.11999997776002,
          "lon" : -120.23000006563962
        },
        "bottom_right" : {
          "lat" : 36.249999976716936,
          "lon" : -109.83000006526709
        }
      }
    }
  }
}
```
</details>

当前，OpenSearch通过API支持GeoShape聚合，但在OpenSearch仪表板可视化中不支持GEOSHAPE聚合。如果您想查看为可视化实施的GeoShape聚合，请访问相关的[Github问题](https://github.com/opensearch-project/dashboards-maps/issues/250)。
{:.note}


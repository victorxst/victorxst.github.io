---
layout: default
title: Geohex网格
parent: 桶聚合
grand_parent: 聚合
nav_order: 85
redirect_from:
  - /aggregations/geohexgrid/
  - /query-dsl/aggregations/geohexgrid/
  - /query-dsl/aggregations/bucket/geohex-grid/
---

# Geohex网格聚集

六边形分层地理空间索引系统（H3）将地球区域划分为可识别的六角形-形状的细胞。

H3网格系统在接近应用程序中效果很好，因为它克服了Geohash的非局限性-统一分区。GeoHash编码纬度和经度对，导致两极附近的分区明显较小，并且在赤道附近的经度程度一定程度。但是，H3网格系统的失真较低，限制为5个分区122。-使用区域（例如，在海洋中部），使基本区域无误差。因此，基于H3网格系统的文档分组比GeoHash网格提供了更好的聚合。

Geohex网格聚集组[地理点]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geo-point/) 进入网格单元进行地理分析。每个网格单元对应于[H3细胞](https://h3geo.org/docs/core-library/h3Indexing/#h3-cell-indexp) 并使用[H3Index表示](https://h3geo.org/docs/core-library/h3Indexing/#h3index-representation)。

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
{: .note}

## 低的-精度请求

低点-精确要求将所有三个文件都结合在一起：

```json
GET national_parks/_search
{
  "aggregations": {
    "grouped": {
      "geohex_grid": {
        "field": "location",
        "precision": 1
      }
    }
  }
}
```
{% include copy-curl.html %}

您可以使用`GET` 或者`POST` HTTP方法用于GeoHex网格聚合查询。
{: .note}

响应组文件2和3在一起，因为它们足够近以在一个网格单元格中放置：

```json
{
  "took" : 4,
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
          "location" : "44.42, -110.59"
        }
      },
      {
        "_index" : "national_parks",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "name" : "Yosemite National Park",
          "location" : "37.87, -119.53"
        }
      },
      {
        "_index" : "national_parks",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "name" : "Death Valley National Park",
          "location" : "36.53, -116.93"
        }
      }
    ]
  },
  "aggregations" : {
    "grouped" : {
      "buckets" : [
        {
          "key" : "8129bffffffffff",
          "doc_count" : 2
        },
        {
          "key" : "8128bffffffffff",
          "doc_count" : 1
        }
      ]
    }
  }
}
```

## 高的-精度请求

现在跑高-精确请求：

```json
GET national_parks/_search
{
  "aggregations": {
    "grouped": {
      "geohex_grid": {
        "field": "location",
        "precision": 6
      }
    }
  }
}
```
{% include copy-curl.html %}

由于粒度较高，这三个文档都是单独贴上的：

```json
{
  "took" : 5,
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
          "location" : "44.42, -110.59"
        }
      },
      {
        "_index" : "national_parks",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "name" : "Yosemite National Park",
          "location" : "37.87, -119.53"
        }
      },
      {
        "_index" : "national_parks",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "name" : "Death Valley National Park",
          "location" : "36.53, -116.93"
        }
      }
    ]
  },
  "aggregations" : {
    "grouped" : {
      "buckets" : [
        {
          "key" : "8629ab6dfffffff",
          "doc_count" : 1
        },
        {
          "key" : "8629857a7ffffff",
          "doc_count" : 1
        },
        {
          "key" : "862896017ffffff",
          "doc_count" : 1
        }
      ]
    }
  }
}
```

## 过滤请求

高的-精确请求是资源密集的，因此我们建议使用诸如此类的过滤器`geo_bounding_box` 限制地理区域。例如，以下查询应用过滤器以限制搜索区域：

```json
GET national_parks/_search
{
  "size" : 0,  
  "aggregations": {
    "filtered": {
      "filter": {
        "geo_bounding_box": {
          "location": {
            "top_left": "38, -120",
            "bottom_right": "36, -116"
          }
        }
      },
      "aggregations": {
        "grouped": {
          "geohex_grid": {
            "field": "location",
            "precision": 6
          }
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

响应包含两个文档`geo_bounding_box` 边界：

```json
{
  "took" : 4,
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
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "filtered" : {
      "doc_count" : 2,
      "grouped" : {
        "buckets" : [
          {
            "key" : "8629ab6dfffffff",
            "doc_count" : 1
          },
          {
            "key" : "8629857a7ffffff",
            "doc_count" : 1
          }
        ]
      }
    }
  }
}
```

您还可以通过在界限中提供边界信封的坐标来限制地理区域`bounds` 范围。两个都`bounds` 和`geo_bounding_box` 可以在任何一个中指定坐标[地理点格式]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geo-point#formats)。以下查询使用井-已知文本（WKT）"POINT(`longitude` `latitude`)" 格式`bounds` 范围：

```json
GET national_parks/_search
{
  "size": 0,
  "aggregations": {
    "grouped": {
      "geohex_grid": {
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
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "grouped" : {
      "buckets" : [
        {
          "key" : "8629ab6dfffffff",
          "doc_count" : 1
        },
        {
          "key" : "8629857a7ffffff",
          "doc_count" : 1
        }
      ]
    }
  }
}
```

这`bounds` 参数可以在有或没有的情况下使用`geo_bounding_box` 筛选;这两个参数是独立的，并且可以彼此之间存在任何空间关系。

## 支持的参数

GeoHex网格聚合请求支持以下参数。

范围| 数据类型| 描述
:--- | :--- | :--- 
场地| 细绳| 包含地理点的字段。该字段必须映射为`geo_point` 场地。如果字段包含一个数组，则所有数组值均已汇总。必需的。
精确| 整数| 缩放水平用于确定网格单元以进行铲斗结果。有效值在[0，15]范围内。选修的。默认值为5。
边界| 目的| 用于过滤地理点的边界框。边界框由鞋面定义-左右-右顶点。这些顶点在以下格式之一中指定为地理点：<br>- 具有纬度和经度的对象<br>- 阵列[`longitude`，`latitude`](format<br>- A string in the "`latitude`,`longitude`" format<br>- A geohash <br>- WKT<br> See the [geopoint formats]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geo-point#formats) 用于格式化示例。选修的。
尺寸| 整数| 返回的最大存储桶数。当水桶比`size`，OpenSearch返回带有更多文档的存储桶。选修的。默认值为10,000。
shard_size| 整数| 从每个碎片返回的最大存储桶数。选修的。默认值为最大（10，`size` ＆middot;碎片数），这提供了更准确的更优先级存储桶的计数


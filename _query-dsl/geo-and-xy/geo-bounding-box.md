---
layout: default
title: 地理-边界框
parent: 地理和XY查询
grand_parent: 查询DSL
nav_order: 10
redirect_from:
  - /opensearch/query-dsl/geo-and-xy/geo-bounding-box/
  - /query-dsl/query-dsl/geo-and-xy/geo-bounding-box/
---

# 地理-边界框查询

搜索包含的文档[地理点]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geo-point/) 字段，使用地理-边界框查询。地理-边界框查询返回其地理点的文档，其地理点位于查询中指定的边界框中。如果至少一个地理点在边界框中，则具有多个地理点的文档匹配查询。

## 例子

您可以使用地理-边界框查询以搜索包含地理点的文档。

用`point` 字段映射为`geo_point`：

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

索引三个地理点作为具有纬度和纵向的对象：

```json
PUT testindex1/_doc/1
{
  "point": { 
    "lat": 74.00,
    "lon": 40.71
  }
}

PUT testindex1/_doc/2
{
  "point": { 
    "lat": 72.64,
    "lon": 22.62
  } 
}

PUT testindex1/_doc/3
{
  "point": { 
    "lat": 75.00,
    "lon": 28.00
  }
}
```

搜索所有文档并过滤其点位于查询中定义的矩形内的文档：

```json
GET testindex1/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "point": {
            "top_left": {
              "lat": 75,
              "lon": 28
            },
            "bottom_right": {
              "lat": 73,
              "lon": 41
            }
          }
        }
      }
    }
  }
}
```

响应包含匹配文档：

```json
{
  "took" : 20,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "testindex1",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "point" : {
            "lat" : 74.0,
            "lon" : 40.71
          }
        }
      }
    ]
  }
}
```

前面的响应不包括文档的地理位置`"lat": 75.00, "lon": 28.00` 由于地理点的有限[精确](#precision)。
{: .note}

## 精确

地理点坐标始终在索引时间下降。在查询时间，边界框的上限被舍入，下边界被舍入。因此，由于舍入错误，结果可能不包含在边界框的下部和左边缘上的地理点的文档。另一方面，即使在边界不在边界之外，位于边界框的上和右边缘的地理点也可能包括。舍入错误小于4.20且时间；10 <sup>＆minus; 8 </sup>纬度度，小于8.39＆times;10 <sup>＆minus; 8 </sup>度的经度（约1厘米）。

## 指定边界框

您可以通过提供其顶点坐标的以下任何组合来指定边界框：

- `top_left` 和`bottom_right`
- `top_right` 和`bottom_left`
- `top`，`left`，`bottom`， 和`right`

以下示例显示了如何使用该框来指定边界框`top`，`left`，`bottom`， 和`right` 坐标：

```json
GET testindex1/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "point": {
            "top": 75,
            "left": 28,
            "bottom": 73,
            "right": 41            
          }
        }
      }
    }
  }
}
```

## 请求字段

地理-边界框查询接受以下字段。

场地| 数据类型| 描述
:--- | :--- | :--- 
`_name` | 细绳| 过滤器的名称。选修的。
`validation_method` | 细绳| 验证方法。有效值是`IGNORE_MALFORMED` （接受具有无效坐标的地理点），`COERCE` （尝试将坐标胁到有效值），并且`STRICT` （当坐标无效时返回错误）。默认为`STRICT`。
`type` | 细绳| 指定如何执行过滤器。有效值是`indexed` （索引过滤器）和`memory` （在内存中执行过滤器）。默认为`memory`。
`ignore_unmapped` | 布尔| 指定是否忽略未绘制的字段。如果设置为`true`，该查询不会返回任何具有未上限字段的文档。如果设置为`false`，当字段未绘制时，会抛出一个例外。默认为`false`。

## 接受格式

您可以在任何任何中指定边界框顶点的坐标[格式]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/geo-point#formats) 地理点字段类型接受。

### 使用GeoHash指定边界框

如果您使用地理什（Geohash）指定边界框，则将GeoHash视为矩形。上层-边界框的左顶点对应于鞋面-左顶点`top_left` Geohash，较低-边界框的右顶点对应于下部-右顶点`bottom_right` Geohash。

下面的示例显示了如何使用GeoHash指定与以前的示例相同的边界框：

```json
GET testindex1/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "point": {
            "top_left": "ut7ftjkfxm34",
            "bottom_right": "uuvpkcprc4rc"
          }
        }
      }
    }
  }
}
```

要指定一个覆盖GeoHash整个区域的边界框`top_left` 和`bottom_right` 边界框的参数：

```json
GET testindex1/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "point": {
            "top_left": "ut",
            "bottom_right": "ut"
          }
        }
      }
    }
  }
}
```

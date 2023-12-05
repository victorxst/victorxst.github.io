---
layout: default
title: xy
parent: 地理和XY查询
grand_parent: 查询DSL
nav_order: 50

redirect_from: 
  - /opensearch/query-dsl/geo-and-xy/xy/
  - /query-dsl/query-dsl/geo-and-xy/xy/
---

# xy查询

搜索包含的文档[xy点]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/xy-point) 和[XY形状]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/xy-shape) 字段，使用XY查询。

## 空间关系

当您向XY查询提供XY形状时，使用以下空间关系与所提供的形状匹配XY字段。

关系| 描述| 支持XY字段类型
：--- | ：--- | ：--- 
`INTERSECTS` | （默认值）匹配其XY点或XY形状与查询中提供的形状相交的文档。| `xy_point`，，，，`xy_shape`
`DISJOINT` | 匹配XY形状的文档与查询中提供的形状并不相交。| `xy_shape`
`WITHIN` | 匹配其XY形状完全在查询中提供的形状之内的文档。| `xy_shape`
`CONTAINS` | 匹配的文档的XY形状完全包含查询中提供的形状。| `xy_shape`

以下示例说明了搜索包含XY形状的文档。要了解如何搜索包含XY点的文档，请参阅[查询xy点](#querying-xy-points) 部分。

## 在xy查询中定义形状

您可以通过在查询时间提供新的形状定义或引用形状的名称pre来定义xy查询中的形状-在另一个索引中索引。

### 使用新形状定义

要为XY查询提供新形状，请在`xy_shape` 场地。

以下示例说明了搜索具有XY形状的文档，该文档符合查询时间定义的XY形状。

首先，创建索引并映射`geometry` 字段作为一个`xy_shape`：

```json
PUT testindex
{
  "mappings": {
    "properties": {
      "geometry": {
        "type": "xy_shape"
      }
    }
  }
}
```

带有点的文档和带有多边形的文档：

```json
PUT testindex/_doc/1
{
  "geometry": { 
    "type": "point",
    "coordinates": [0.5, 3.0]
  }
}

PUT testindex/_doc/2
{
  "geometry" : {
    "type" : "polygon",
    "coordinates" : [
      [[2.5, 6.0],
      [0.5, 4.5], 
      [1.5, 2.0], 
      [3.5, 3.5],
      [2.5, 6.0]]
    ]
  }
}
```

定义[`envelope`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/xy-shape#envelope)＆mdash;一个边界矩形`[[minX, maxY], [maxX, minY]]` 格式。搜索具有XY点或形状相交的文档，这些文档相交的包封件：

```json
GET testindex/_search
{
  "query": {
    "xy_shape": {
      "geometry": {
        "shape": {
          "type": "envelope",
          "coordinates": [ [ 0.0, 6.0], [ 4.0, 2.0] ]
        },
        "relation": "WITHIN"
      }
    }
  }
}
```

下图描述了示例。点和多边形都在边界包膜内。

<img src="{{site.url}}{{site.baseurl}}/images/xy_query.png" alt="xy shape query" width="250">


响应包含两个文档：

```json
{
  "took" : 363,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.0,
    "hits" : [
      {
        "_index" : "testindex",
        "_id" : "1",
        "_score" : 0.0,
        "_source" : {
          "geometry" : {
            "type" : "point",
            "coordinates" : [
              0.5,
              3.0
            ]
          }
        }
      },
      {
        "_index" : "testindex",
        "_id" : "2",
        "_score" : 0.0,
        "_source" : {
          "geometry" : {
            "type" : "polygon",
            "coordinates" : [
              [
                [
                  2.5,
                  6.0
                ],
                [
                  0.5,
                  4.5
                ],
                [
                  1.5,
                  2.0
                ],
                [
                  3.5,
                  3.5
                ],
                [
                  2.5,
                  6.0
                ]
              ]
            ]
          }
        }
      }
    ]
  }
}
```

### 使用pre-索引形状定义

构建XY查询时，您还可以引用形状的名称-在另一个索引中索引。使用此方法，您可以在索引时间定义XY形状，并通过名称参考它，在此处提供以下参数`indexed_shape` 目的。

范围| 描述
：--- | ：---
`index` | 包含PRE的索引的名称-索引形状。
`id` | 包含PRE的文档ID-索引形状。
`path` | 包含PRE的字段的字段名称-索引形状作为路径。

以下示例说明了引用形状的名称-在另一个索引中索引。在此示例中，索引`pre-indexed-shapes` 包含定义边界的形状，索引`testindex` 包含对这些边界进行检查的形状。

首先，创建一个索引`pre-indexed-shapes` 并映射`geometry` 此索引的字段`xy_shape`：

```json
PUT pre-indexed-shapes
{
  "mappings": {
    "properties": {
      "geometry": {
        "type": "xy_shape"
      }
    }
  }
}
```

索引一个指定边界并命名的信封`rectangle`：

```json
PUT pre-indexed-shapes/_doc/rectangle
{
  "geometry": {
    "type": "envelope",
    "coordinates" : [ [ 0.0, 6.0], [ 4.0, 2.0] ]
  }
}
```

索引一个带有点的文档和带有多边形的文档索引索引`testindex`：

```json
PUT testindex/_doc/1
{
  "geometry": { 
    "type": "point",
    "coordinates": [0.5, 3.0]
  }
}

PUT testindex/_doc/2
{
  "geometry" : {
    "type" : "polygon",
    "coordinates" : [
      [[2.5, 6.0],
      [0.5, 4.5], 
      [1.5, 2.0], 
      [3.5, 3.5],
      [2.5, 6.0]]
    ]
  }
}
```

搜索具有相交形状的文档`rectangle` 在索引中`testindex` 使用过滤器：

```json
GET testindex/_search
{
  "query": {
    "bool": {
      "filter": {
        "xy_shape": {
          "geometry": {
            "indexed_shape": {
              "index": "pre-indexed-shapes",
              "id": "rectangle",
              "path": "geometry"
            }
          }
        }
      }
    }
  }
}
```

前面的查询使用默认空间关系`INTERSECTS` 并返回点和多边形：

```json
{
  "took" : 26,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.0,
    "hits" : [
      {
        "_index" : "testindex",
        "_id" : "1",
        "_score" : 0.0,
        "_source" : {
          "geometry" : {
            "type" : "point",
            "coordinates" : [
              0.5,
              3.0
            ]
          }
        }
      },
      {
        "_index" : "testindex",
        "_id" : "2",
        "_score" : 0.0,
        "_source" : {
          "geometry" : {
            "type" : "polygon",
            "coordinates" : [
              [
                [
                  2.5,
                  6.0
                ],
                [
                  0.5,
                  4.5
                ],
                [
                  1.5,
                  2.0
                ],
                [
                  3.5,
                  3.5
                ],
                [
                  2.5,
                  6.0
                ]
              ]
            ]
          }
        }
      }
    ]
  }
}
```

## 查询xy点

您还可以使用XY查询搜索包含XY点的文档。

与`point` 作为`xy_point`：

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

索引三分：

```json
PUT testindex1/_doc/1
{
  "point": "1.0, 1.0" 
}

PUT testindex1/_doc/2
{
  "point": "2.0, 0.0" 
}

PUT testindex1/_doc/3
{
  "point": "-2.0, 2.0" 
}
```

搜索中心在（0，0）中的圆圈内的点，半径为2：

```json
GET testindex1/_search
{
  "query": {
    "xy_shape": {
      "point": {
        "shape": {
          "type": "circle",
          "coordinates": [0.0, 0.0],
          "radius": 2
        }
      }
    }
  }
}
```

xy点仅支持默认值`INTERSECTS` 空间关系，因此您无需提供`relation` 范围。
{： 。笔记}

下图描述了示例。点1和2在圆圈内，点3在圆形之外。

<img src="{{site.url}}{{site.baseurl}}/images/xy_query_point.png" alt="xy point query" width="300">

响应返回文档1和2：

```json
{
  "took" : 575,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.0,
    "hits" : [
      {
        "_index" : "testindex1",
        "_id" : "1",
        "_score" : 0.0,
        "_source" : {
          "point" : "1.0, 1.0"
        }
      },
      {
        "_index" : "testindex1",
        "_id" : "2",
        "_score" : 0.0,
        "_source" : {
          "point" : "2.0, 0.0"
        }
      }
    ]
  }
}
```

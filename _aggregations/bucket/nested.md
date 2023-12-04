---
layout: default
title: 嵌套
parent: 桶聚合
grand_parent: 聚合
nav_order: 140
redirect_from:
  - /query-dsl/aggregations/bucket/nested/
---

# 嵌套聚集

这`nested` 聚合使您可以在嵌套对象内的字段上进行聚合。这`nested` 类型是对象数据类型的专业版本，该版本允许以可以独立于彼此查询的方式索引对象数组

与`object` 类型，所有数据都存储在同一文档中，因此搜索的匹配可以在子文档中进行。例如，想象一个`logs` 索引`pages` 映射为一个`object` 数据类型：

```json
PUT logs/_doc/0
{
  "response": "200",
  "pages": [
    {
      "page": "landing",
      "load_time": 200
    },
    {
      "page": "blog",
      "load_time": 500
    }
  ]
}
```
{% include copy-curl.html %}

OpenSearch合并所有子-看起来像这样的实体关系的属性：

```json
{
  "logs": {
    "pages": ["landing", "blog"],
    "load_time": ["200", "500"]
  }
}
```

因此，如果您想搜索此索引`pages=landing` 和`load_time=500`，即使`load_time` 着陆的价值为200。

如果您想确保这样的十字架-对象匹配不会发生，将字段映射为`nested` 类型：

```json
PUT logs
{
  "mappings": {
    "properties": {
      "pages": {
        "type": "nested",
        "properties": {
          "page": { "type": "text" },
          "load_time": { "type": "double" }
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

嵌套文档允许您索引相同的JSON文档，但会将您的页面保存在单独的Lucene文档中，仅进行搜索`pages=landing` 和`load_time=200` 返回预期的结果。在内部，嵌套对象将数组中的每个对象索引为一个单独的隐藏文档，这意味着每个嵌套对象可以独立查询其他对象。

您必须指定相对于父母的嵌套路径，其中包含嵌套文档：


```json
GET logs/_search
{
  "query": {
    "match": { "response": "200" }
  },
  "aggs": {
    "pages": {
      "nested": {
        "path": "pages"
      },
      "aggs": {
        "min_load_time": { "min": { "field": "pages.load_time" } }
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
  "pages" : {
    "doc_count" : 2,
    "min_price" : {
      "value" : 200.0
    }
  }
}
```


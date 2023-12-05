---
layout: default
title: 存在
parent: 术语级查询
grand_parent: 查询DSL
nav_order: 10
---

# 存在查询

使用`exists` 查询以搜索包含特定字段的文档。

在以下任何情况下，文档字段都不存在索引值：

- 该领域有`"index" : false` 在映射中指定。
- 来源JSON中的字段是`null` 或者`[]`。
- 场值的长度超过`ignore_above` 设置在映射中。
- 现场值畸形，并且`ignore_malformed` 在映射中定义。

在以下任何情况下，文档字段都将存在索引值：

- 该值是一个包含一个或多个空元素的数组，一个或多个非元素-空元素（例如，`["one", null]`）。
- 值是一个空字符串（`""` 或者`"-"`）。
- 值是自定义`null_value`，如字段映射中所定义。


## 例子

例如，考虑包含以下两个文档的索引：

```json
PUT testindex/_doc/1
{
  "title": "The wind rises"
}
```
{％包含副本-curl.html％}

```json
PUT testindex/_doc/2
{
  "title": "Gone with the wind",
  "description": "A 1939 American epic historical film"
}
```
{％包含副本-curl.html％}

以下查询搜索包含的文档`description` 场地：

```json
GET testindex/_search
{
  "query": {
    "exists": {
      "field": "description"
    }
  }
}
```
{％包含副本-curl.html％}

响应包含匹配文档：

```json
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 1,
        "_source": {
          "title": "Gone with the wind",
          "description": "A 1939 American epic historical film"
        }
      }
    ]
  }
}
```

## 查找缺少索引值的文档

要查找缺少索引值的文档，您可以使用`must_not` [布尔查询]({{site.url}}{{site.baseurl}}/query-dsl/compound/bool/) 与内心`exists` 询问。例如，以下请求搜索文档`description` 缺少字段：

```json
GET testindex/_search
{
  "query": {
    "bool": {
      "must_not": {
        "exists": {
          "field": "description"
        }
      }
    }
  }
}
```
{％包含副本-curl.html％}

响应包含匹配文档：

```json
{
  "took": 19,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0,
        "_source": {
          "title": "The wind rises"
        }
      }
    ]
  }
}
```

## 参数

查询接受字段的名称（`<field>`）作为顶部-级别参数。


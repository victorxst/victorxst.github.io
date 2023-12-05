---
layout: default
title: 析取最大值
parent: 复合查询
grand_parent: 查询DSL
nav_order: 50
redirect_from:
  - /query-dsl/query-dsl/compound/disjunction-max/
---

# 析取最大值

max的分离（`dis_max`）查询返回与一个或多个查询子句匹配的任何文档。对于匹配多个查询子句的文档，相关得分设置为所有匹配查询子句的最高相关性分数。

当返回文档的相关性分数相同时，您可以使用`tie_breaker` 参数可以给匹配多个查询子句的文档提供更多权重。

## 例子

考虑一个带有两个文档的索引，您索引如下：

```json
PUT testindex1/_doc/1
{
  "title": " The Top 10 Shakespeare Poems",
  "description": "Top 10 sonnets of England's national poet and the Bard of Avon"
}
```
{% include copy-curl.html %}

```json
PUT testindex1/_doc/2
{
  "title": "Sonnets of the 16th Century",
  "body": "The poems written by various 16-th century poets"
}
```
{% include copy-curl.html %}

用一个`dis_max` 查询搜索包含单词的文档"Shakespeare poems"：

```json
GET testindex1/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "title": "Shakespeare poems" }},
        { "match": { "body":  "Shakespeare poems" }}
      ]
    }
  }            
}
```
{% include copy-curl.html %}

响应包含两个文档：

```json
{
  "took": 8,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 1.3862942,
    "hits": [
      {
        "_index": "testindex1",
        "_id": "1",
        "_score": 1.3862942,
        "_source": {
          "title": " The Top 10 Shakespeare Poems",
          "description": "Top 10 sonnets of England's national poet and the Bard of Avon"
        }
      },
      {
        "_index": "testindex1",
        "_id": "2",
        "_score": 0.2876821,
        "_source": {
          "title": "Sonnets of the 16th Century",
          "body": "The poems written by various 16-th century poets"
        }
      }
    ]
  }
}
```

## 参数

下表列出了所有顶部-支持的级别参数`dis_max` 查询。

范围| 描述
:--- | :---
`queries` | 用于匹配文档的一个或多个查询子句的数组。文档必须匹配至少一个查询条款才能在结果中返回。如果文档匹配多个查询子句，则将相关性得分设置为所有匹配查询子句的最高相关性分数。必需的。
`tie_breaker` | 浮动-点因子在0到1.0之间，用于给匹配多个查询子句的文档提供更大的权重。在这种情况下，使用以下算法计算文档的相关性评分：从所有匹配的查询子句中获取最高相关性得分，将所有其他匹配条款的分数乘以乘以`tie_breaker` 价值，并将相关得分添加在一起，使它们正常化。选修的。默认值为0（仅表示最高分数计数）。


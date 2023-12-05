---
layout: default
title: 提升
parent: 复合查询
grand_parent: 查询DSL
nav_order: 30
redirect_from:
  - /query-dsl/query-dsl/compound/boosting/
---

# 增强查询

如果您要搜索这个单词"pitcher"，您的结果可能与棒球运动员或液体容器有关。要在棒球的背景下进行搜索，您可能需要完全排除包含单词的结果"glass" 或者"water" 通过使用`must_not` 条款。但是，如果您想保留这些结果，但可以将它们降级为相关性，则可以这样做`boosting` 查询。

A`boosting` 查询返回匹配的文档`positive` 询问。在这些文件中，也与`negative` 相关性评分较低（它们的相关得分乘以负增长因子）。

## 例子

考虑一个带有两个文档的索引，您索引如下：

```json
PUT testindex/_doc/1
{
  "article_name": "The greatest pitcher in baseball history"
}
```

```json
PUT testindex/_doc/2
{
  "article_name": "The making of a glass pitcher"
}
```

使用以下匹配查询搜索包含单词的文档"pitcher"：

```json
GET testindex/_search
{
  "query": {
    "match": {
      "article_name": "pitcher"
    }
  }
}
```

两个返回的文档的相关性得分相同：

```json
{
  "took": 5,
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
    "max_score": 0.18232156,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.18232156,
        "_source": {
          "article_name": "The greatest pitcher in baseball history"
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.18232156,
        "_source": {
          "article_name": "The making of a glass pitcher"
        }
      }
    ]
  }
}
```

现在使用以下`boosting` 查询以搜索包含单词的文档"pitcher" 但是降低了包含单词的文档"glass"，"crystal"， 或者"water"：

```json
GET testindex/_search
{
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "article_name": "pitcher"
        }
      },
      "negative": {
        "match": {
          "article_name": "glass crystal water"
        }
      },
      "negative_boost": 0.1
    }
  }
}
```
{% include copy-curl.html %}

两个文档仍然返回，但文档带有单词"glass" 相关得分比以前的情况低10倍：

```json
{
  "took": 13,
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
    "max_score": 0.18232156,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.18232156,
        "_source": {
          "article_name": "The greatest pitcher in baseball history"
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.018232157,
        "_source": {
          "article_name": "The making of a glass pitcher"
        }
      }
    ]
  }
}
```

## 参数

下表列出了所有顶部-支持的级别参数`boosting` 查询。

范围| 描述
:--- | :---
`positive` | 必须在结果中返回文档必须匹配的查询。必需的。
`negative` | 如果结果中的文档与此查询匹配，则通过乘以其原始相关性分数来降低其相关性评分（由`positive` 查询）`negative_boost` 范围。必需的。
`negative_boost` | 浮动-为了减少与之匹配的文档的相关性，将原始相关得分乘以0到1.0之间的点因子。`negative` 询问。必需的。


---
layout: default
title: Rank字段类型
nav_order: 60
has_children: false
grand_parent: 支持的字段类型
redirect_from:
  - /opensearch/supported-field-types/rank/
  - /field-types/rank/
---

# 等级字段类型

下表列出了OpenSearch支持的所有等级字段类型。

字段数据类型| 描述
:--- | :---  
[`rank_feature`](#rank-feature) | 提高或降低文档的相关性得分。
[`rank_features`](#rank-features) | 提高或降低文档的相关性得分。当功能列表稀疏时使用。

可以查询等级功能和等级功能字段[等级特征查询](#rank-feature-query) 仅有的。他们不支持汇总或分类。
{: .note}

## 排名功能

等级功能字段类型使用正面[漂浮]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/numeric/) 提高或降低文档中相关性评分的价值`rank_feature` 询问。默认情况下，此值可以提高相关性分数。要降低相关得分，请设置可选的`positive_score_impact` 参数为false。

### 例子

创建具有等级功能字段的映射：

```json
PUT chessplayers
{
  "mappings": {
    "properties": {
      "name" : {
        "type" : "text"
      },
      "rating": {
        "type": "rank_feature" 
      },
      "age": {
        "type": "rank_feature",
        "positive_score_impact": false 
      }
    }
  }
}
```
{% include copy-curl.html %}

带有rank_feature字段的索引三个文档，可以提高分数（`rating`）和一个降低分数的rank_feature字段（`age`）：

```json
PUT testindex1/_doc/1
{
  "name" : "John Doe",
  "rating" : 2554,
  "age" : 75
}
```
{% include copy-curl.html %}

```json
PUT testindex1/_doc/2
{
  "name" : "Kwaku Mensah",
  "rating" : 2067,
  "age": 10
}
```
{% include copy-curl.html %}

```json
PUT testindex1/_doc/3
{
  "name" : "Nikki Wolf",
  "rating" : 1864,
  "age" : 22
}
```
{% include copy-curl.html %}

## 等级功能查询

使用等级功能查询，您可以按评级，年龄或评级和年龄对玩家进行排名。如果您通过评分对球员进行排名，则更高-评分玩家的相关性得分将更高。如果您按年龄对球员进行排名，那么年轻的球员的相关性得分将更高。

使用等级功能查询根据年龄和评分搜索玩家：

```json
GET chessplayers/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "rank_feature": {
            "field": "rating"
          }
        },
        {
          "rank_feature": {
            "field": "age"
          }
        }
      ]
    }
  }
}
```
{% include copy-curl.html %}

当年龄和评分都排名时，年轻的球员和排名更高的球员都会更好：

```json
{
  "took" : 2,
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
    "max_score" : 1.2093145,
    "hits" : [
      {
        "_index" : "chessplayers",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.2093145,
        "_source" : {
          "name" : "Kwaku Mensah",
          "rating" : 1967,
          "age" : 10
        }
      },
      {
        "_index" : "chessplayers",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 1.0150313,
        "_source" : {
          "name" : "Nikki Wolf",
          "rating" : 1864,
          "age" : 22
        }
      },
      {
        "_index" : "chessplayers",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.8098284,
        "_source" : {
          "name" : "John Doe",
          "rating" : 2554,
          "age" : 75
        }
      }
    ]
  }
}
```

## 等级功能

等级功能字段类型与等级功能字段类型相似，但更适合稀疏功能列表。等级功能字段可以索引数字特征向量，后来用于提高或降低文档的相关性分数`rank_feature` 查询。

### 例子

创建具有等级功能字段的映射：

```json
PUT testindex1
{
  "mappings": {
    "properties": {
      "correlations": {
        "type": "rank_features" 
      }
    }
  }
}
```
{% include copy-curl.html %}

要索引具有等级功能字段的文档，请使用带有字符串键和正浮点值的哈希图：

```json
PUT testindex1/_doc/1
{
  "correlations": { 
    "young kids" : 1,
    "older kids" : 15,
    "teens" : 25.9
  }
}
```
{% include copy-curl.html %}

```json
PUT testindex1/_doc/2
{
  "correlations": {
    "teens": 10,
    "adults": 95.7
  }
}
```
{% include copy-curl.html %}

使用等级功能查询查询文档：

```json
GET testindex1/_search
{
  "query": {
    "rank_feature": {
      "field": "correlations.teens"
    }
  }
}
```
{% include copy-curl.html %}

响应按相关得分进行排名：

```json
{
  "took" : 123,
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
    "max_score" : 0.6258503,
    "hits" : [
      {
        "_index" : "testindex1",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.6258503,
        "_source" : {
          "correlations" : {
            "young kids" : 1,
            "older kids" : 15,
            "teens" : 25.9
          }
        }
      },
      {
        "_index" : "testindex1",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 0.39263803,
        "_source" : {
          "correlations" : {
            "teens" : 10,
            "adults" : 95.7
          }
        }
      }
    ]
  }
}
```

等级特征和等级特征字段使用前九个重要位来精确，导致相对误差约为0.4％。值以2 <sup> -8 </sup> = 0.00390625的相对精度存储。
{: .note}


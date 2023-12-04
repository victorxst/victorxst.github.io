---
layout: default
title: 键入时搜索
nav_order: 53
has_children: false
parent: 自动完成字段类型
grand_grand_parent: 支持的字段类型
redirect_from:
  - /opensearch/supported-field-types/search-as-you-type/
  - /field-types/search-as-you-type/
---

# 搜索-作为-你-类型字段类型

搜索-作为-你-类型字段类型提供搜索-作为-你-使用前缀和Infix完成键入功能。

## 例子

映射搜索-作为-你-类型字段创建n-该领域的克子场，其中n在范围内[2，`max_shingle_size`]。此外，它创建了一个索引前缀子字段。

通过搜索创建映射-作为-你-类型字段：

```json
PUT books
{
  "mappings": {
    "properties": {
      "suggestions": {
        "type": "search_as_you_type"
      }
    }
  }
}
```
{% include copy-curl.html %}

除了`suggestions` 字段，这创造了`suggestions._2gram`，`suggestions._3gram`， 和`suggestions._index_prefix` 字段。

用搜索索引文档-作为-你-类型字段：

```json
PUT books/_doc/1
{
  "suggestions": "one two three four"
}
```
{% include copy-curl.html %}

要按任何顺序匹配术语，请使用BOOL_PREFIX或MULTI-匹配查询。这些查询对搜索术语按指定顺序高的文档进行排名，高于未订单的文档。

```json
GET books/_search
{
  "query": {
    "multi_match": {
      "query": "tw one",
      "type": "bool_prefix",
      "fields": [
        "suggestions",
        "suggestions._2gram",
        "suggestions._3gram"
      ]
    }
  }
}
```
{% include copy-curl.html %}

响应包含匹配文档：

```json
{
  "took" : 13,
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
        "_index" : "books",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "suggestions" : "one two three four"
        }
      }
    ]
  }
}
```

要按顺序匹配术语，请使用match_phrase_prefix查询：

```json
GET books/_search
{
  "query": {
    "match_phrase_prefix": {
      "suggestions": "two th"
    }
  }
}
```
{% include copy-curl.html %}

响应包含匹配文档：

```json
{
  "took" : 23,
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
    "max_score" : 0.4793051,
    "hits" : [
      {
        "_index" : "books",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.4793051,
        "_source" : {
          "suggestions" : "one two three four"
        }
      }
    ]
  }
}
```

要准确匹配最后一个术语，请使用match_phrase查询：

```json
GET books/_search
{
  "query": {
    "match_phrase": {
      "suggestions": "four"
    }
  }
}
```
{% include copy-curl.html %}

回复：

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
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "books",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "suggestions" : "one two three four"
        }
      }
    ]
  }
}
```

## 参数

下表列出了搜索接受的参数-作为-你-类型字段类型。所有参数都是可选的。

范围| 描述
：--- | ：---
`analyzer` | 用于此字段的分析仪。默认情况下，它将在索引时和搜索时间使用。要在搜索时间覆盖它，请设置`search_analyzer` 范围。默认为`standard` 分析仪，使用语法-基于令牌化，并基于[Unicode文本细分](https://unicode.org/reports/tr29/) 算法。配置根字段和子字段。
`index` | 布尔值指定是否应搜索该字段。默认为`true`。配置根字段和子字段。
`index_options` | 指定要存储在索引中的信息进行搜索和突出显示。有效值：`docs` （仅DOC编号），`freqs` （文档编号和术语频率），`positions` （文档编号，任期频率和期限位置），`offsets` （文档编号，术语频率，术语位置以及启动和最终字符偏移）。默认为`positions`。配置根字段和子字段。
`max_shingle_size` | 一个指定最大n的整数-克大小。有效值在[2，4]范围内。n-要创建的克在范围内[2，`max_shingle_size`]。默认值为3，创建一个2-克和3-公克。更大`max_shingle_size` 值可在更具体的查询中工作得更好，但导致索引尺寸较大。
`norms` | 布尔值指定在计算相关性分数时是否应使用字段长度。配置根字段和n-克子场（默认为`false`）。不配置前缀子字段（在前缀子字段中，`norms` 是`false`）。
`search_analyzer` | 分析仪将在搜索时间使用。默认是在`analyzer` 范围。配置根字段和子字段。
`search_quote_analyzer` | 用短语在搜索时间使用的分析仪。默认是在`analyzer` 范围。配置根字段和子字段。
`similarity` | 用于计算相关性得分的排名算法。默认为`BM25`。配置根字段和子字段。
`store` | 布尔值指定是否应存储字段值，并且可以与_source字段分开检索。默认为`false`。仅配置根字段。
[`term_vector`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/text#term-vector-parameter) | 布尔值指定是否应存储该字段的术语向量。默认为`no`。配置根字段和n-克子场。不配置前缀子字段。


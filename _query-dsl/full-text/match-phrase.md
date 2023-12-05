---
layout: default
title: 匹配短语
parent: 全文查询
grand_parent: 查询DSL
nav_order: 30
---

# 匹配短语查询

使用`match_phrase` 查询以匹配包含指定顺序的精确短语的文档。您可以通过提供添加短语匹配的灵活性`slop` 范围。

这`match_phrase` 查询创建一个[短语查询](https://lucene.apache.org/core/8_9_0/core/org/apache/lucene/search/PhraseQuery.html) 与一系列术语相匹配。

以下示例显示了一个基本`match_phrase` 询问：

```json
GET _search
{
  "query": {
    "match_phrase": {
      "title": "the wind"
    }
  }
}
```
{％包含副本-curl.html％}

要传递其他参数，您可以使用扩展的语法：

```json
GET _search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "the wind",
        "analyzer": "stop"
      }
    }
  }
}
```
{％包含副本-curl.html％}

## 例子

例如，考虑具有以下文档的索引：

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
  "title": "Gone with the wind"
  
}
```
{％包含副本-curl.html％}

下列`match_phrase` 查询搜索短语`wind rises`，一个单词`wind` 接下来是这个词`rises`：

```json
GET testindex/_search
{
  "query": {
    "match_phrase": {
      "title": "wind rises"
    }
  }
}
```
{％包含副本-curl.html％}

响应包含匹配文档：

<详细信息关闭的markdown ="block">
  <summary>
    回复
  </summary>
  {： 。文本-三角洲}


```json
{
  "took": 30,
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
    "max_score": 0.92980814,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.92980814,
        "_source": {
          "title": "The wind rises"
        }
      }
    ]
  }
}
```
</delect>

## 分析仪

默认情况下，当您在`text` 字段，使用与字段关联的索引分析仪分析搜索文本。您可以在`analyzer` 范围。例如，以下查询使用`english` 分析仪：

```json
GET testindex/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "the winds",
        "analyzer": "english"
      }
    }
  }
}
```
{％包含副本-curl.html％}

这`english` 分析仪删除了停止词`the` 并执行茎，产生令牌`wind`。这两个文档都与此令牌匹配，并在结果中返回：

<详细信息关闭的markdown ="block">
  <summary>
    回复
  </summary>
  {： 。文本-三角洲}

```json
{
  "took": 2,
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
    "max_score": 0.19363807,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.19363807,
        "_source": {
          "title": "The wind rises"
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.17225474,
        "_source": {
          "title": "Gone with the wind"
        }
      }
    ]
  }
}
```
</delect>

## 坡

如果您提供`slop` 参数，查询可容忍搜索词的重新排序。SLOP指定查询短语中单词之间允许的其他单词数量。例如，在以下查询中，与文档文本相比，对搜索文本进行了重新排序：

```json
GET _search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "wind rises the",
        "slop": 3
      }
    }
  }
}
```

查询仍然返回匹配文档：

<详细信息关闭的markdown ="block">
  <summary>
    回复
  </summary>
  {： 。文本-三角洲}

```json
{
  "took": 2,
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
    "max_score": 0.44026947,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.44026947,
        "_source": {
          "title": "The wind rises"
        }
      }
    ]
  }
}
```
</delect>

## 空查询

有关可能的空查询的信息，请参阅相应[匹配查询部分]({{site.url}}{{site.baseurl}}/query-dsl/full-text/match/#empty-query)。

## 参数

查询接受字段的名称（`<field>`）作为顶部-级别参数：

```json
GET _search
{
  "query": {
    "match_phrase": {
      "<field>": {
        "query": "text to search for",
        ... 
      }
    }
  }
}
```
{％包含副本-curl.html％}

这`<field>` 接受以下参数。除所有参数外`query` 是可选的。

范围| 数据类型| 描述
：--- | ：--- | ：---
`query` | 细绳| 用于搜索的查询字符串。必需的。
`analyzer` | 细绳| 这[分析仪]({{site.url}}{{site.baseurl}}/analyzers/index/) 用于引导查询字符串文本。默认是索引-指定的时间分析仪`default_field`。如果未针对`default_field`， 这`analyzer` 是索引的默认分析仪。
`slop` | `0` （默认）或正整数| 控制查询中的单词可能会被误解的程度，并且仍然被认为是匹配的程度。来自[Lucene文档](https://lucene.apache.org/core/8_9_0/core/org/apache/lucene/search/PhraseQuery.html#getSlop--)："The number of other words permitted between words in query phrase. For example, to switch the order of two words requires two moves (the first move places the words atop one another), so to permit reorderings of phrases, the slop must be at least two. A value of zero requires an exact match."
`zero_terms_query` | 细绳| 在某些情况下，分析仪从查询字符串中删除了所有术语。例如，`stop` 分析仪从字符串中删除所有术语`an but this`。在这些情况下，`zero_terms_query` 指定是否匹配不匹配文档（`none`）或所有文件（`all`）。有效值是`none` 和`all`。默认为`none`


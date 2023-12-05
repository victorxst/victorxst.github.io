---
layout: default
title: 匹配短语前缀
parent: 全文查询
grand_parent: 查询DSL
nav_order: 40
---

# 匹配短语前缀查询

使用`match_phrase_prefix` 查询以指定按顺序匹配的短语。包含您指定的短语的文档将被返回。该短语中的最后一个术语被解释为前缀，因此任何包含以短语开始的短语和上一项前缀开始的短语的文档都将被返回。

如同[匹配短语]({{site.url}}{{site.baseurl}}/query-dsl/full-text/match-phrase/)，但创造了一个[前缀查询](https://lucene.apache.org/core/8_9_0/core/org/apache/lucene/search/PrefixQuery.html) 在查询字符串中的最后一个学期。

对于差异`match_phrase_prefix` 和`match_bool_prefix` 查询，请参阅[这`match_bool_prefix` 和`match_phrase_prefix` 查询]({{site.url}}{{site.baseurl}}/query-dsl/full-text/match-bool-prefix/#the-match_bool_prefix-and-match_phrase_prefix-queries)。

以下示例显示了一个基本`match_phrase_prefix` 询问：

```json
GET _search
{
  "query": {
    "match_phrase_prefix": {
      "title": "the wind"
    }
  }
}
```
{% include copy-curl.html %}

要传递其他参数，您可以使用扩展的语法：

```json
GET _search
{
  "query": {
    "match_phrase_prefix": {
      "title": {
        "query": "the wind",
        "analyzer": "stop"
      }
    }
  }
}
```
{% include copy-curl.html %}

## 例子

例如，考虑具有以下文档的索引：

```json
PUT testindex/_doc/1
{
  "title": "The wind rises"
}
```
{% include copy-curl.html %}

```json
PUT testindex/_doc/2
{
  "title": "Gone with the wind"
  
}
```
{% include copy-curl.html %}

下列`match_phrase_prefix` 查询搜索整个单词`wind`，然后是一个始于`ri`：

```json
GET testindex/_search
{
  "query": {
    "match_phrase_prefix": {
      "title": "wind ri"
    }
  }
}
```
{% include copy-curl.html %}

响应包含匹配文档：

<详细信息关闭的markdown ="block">
  <summary>
    回复
  </summary>
  {： 。文本-三角洲}

```json
{
  "took": 6,
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
</details>

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
{% include copy-curl.html %}

这`<field>` 接受以下参数。除所有参数外`query` 是可选的。

范围| 数据类型| 描述
:--- | :--- | :---
`query` | 细绳| 用于搜索的查询字符串。必需的。
`analyzer` | 细绳| 这[分析仪]({{site.url}}{{site.baseurl}}/analyzers/index/) 用于引导查询。
`max_expansions` | 正整数|  查询可以扩展的最大术语数量。模糊的查询“扩展为”在指定距离内的许多匹配术语`fuzziness`。然后OpenSearch尝试匹配这些术语。默认为`50`。
`slop` | `0` （默认）或正整数| 控制查询中的单词可能会被误解的程度，并且仍然被认为是匹配的程度。来自[Lucene文档](https://lucene.apache.org/core/8_9_0/core/org/apache/lucene/search/PhraseQuery.html#getSlop--)：“查询短语中单词之间允许的其他单词数量。例如，要切换两个单词的顺序需要两个动作（第一步将单词彼此放在彼此之间），因此要允许对短语进行重新排序，则必须是至少两个。零值需要确切的匹配。


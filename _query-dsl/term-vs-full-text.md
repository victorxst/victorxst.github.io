---
layout: default
title: 术语级查询和全文查询的比较
nav_order: 10
redirect_from:
- /query-dsl/query-dsl/term-vs-full-text
---

# 术语级查询和全文查询的比较

您可以使用两个术语-水平和饱满-文本查询以搜索文本，但是-级别查询通常用于搜索结构化数据，完整-文本查询用于完整-文字搜索。术语之间的主要区别-水平和饱满-文字查询是该术语-级别查询搜索文档的精确指定术语，而已满-文本查询分析查询字符串。下表总结了术语之间的差异-水平和饱满-文本查询。

| | 学期-级查询| 满的-文本查询
:--- | :--- | :---
*描述*| 学期-级别查询答案哪些文档匹配查询。| 满的-文本查询回答文档的匹配程度。
*分析仪*| 搜索词未分析。这意味着术语查询搜索您的搜索词。| 搜索术语由相同的分析仪分析，该分析仪在索引时用于特定文档字段。这意味着您的搜索词与文档字段相同的分析过程。
*关联*| 学期-级别查询只需返回匹配的文档而无需根据相关得分对其进行排序。他们仍然计算相关得分，但是对于返回的所有文档而言，此分数相同。| 满的-文本查询计算每场比赛的相关得分，并通过降低相关性顺序对结果进行排序。
*用例*| 使用术语-当您想匹配精确值（例如数字，日期或标签）时，级别查询不需要相关性对匹配进行排序。| 完整使用-在考虑了壳体和茎变体等因素之后，文本查询以匹配文本字段并按相关性进行排序。

OpenSearch使用BM25排名算法来计算相关性得分。要了解更多，请参阅[Okapi BM25](https://en.wikipedia.org/wiki/Okapi_BM25)。
{:.note}

## 我应该用一个完整的-文字或术语-级查询？

澄清完整的区别-文字和术语-级别查询，请考虑以下两个搜索特定文本短语的示例。莎士比亚的完整作品在OpenSearch集群中索引。

### 示例：短语搜索

在此示例中，您将搜索莎士比亚的完整作品中的短语"To be, or not to be" 在里面`text_entry` 场地。

首先，使用**学期-级查询** 对于此搜索：

```json
GET shakespeare/_search
{
  "query": {
    "term": {
      "text_entry": "To be, or not to be"
    }
  }
}
```

响应不包含匹配，由零表示`hits`：

```json
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}
```

这是因为在倒置索引中从字面上搜索了术语“要或不存在”一词，在该索引中，仅存储了文本字段的分析值。学期-级别查询不适合搜索分析的文本字段，因为它们通常会产生意外的结果。使用文本数据时，使用术语-仅针对映射的字段查询`keyword`。

现在使用一个**满的-文本查询**：

```json
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": "To be, or not to be"
    }
  }
}
```

搜索查询“待或不在”被分析并将令牌分析为一系列令牌`text_entry` 文件字段。完整-文本查询在搜索查询和`text_entry` 所有文档的字段，然后按相关得分对结果进行分类：

```json
{
  "took" : 19,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : 17.419369,
    "hits" : [
      {
        "_index" : "shakespeare",
        "_id" : "34229",
        "_score" : 17.419369,
        "_source" : {
          "type" : "line",
          "line_id" : 34230,
          "play_name" : "Hamlet",
          "speech_number" : 19,
          "line_number" : "3.1.64",
          "speaker" : "HAMLET",
          "text_entry" : "To be, or not to be: that is the question:"
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "109930",
        "_score" : 14.883024,
        "_source" : {
          "type" : "line",
          "line_id" : 109931,
          "play_name" : "A Winters Tale",
          "speech_number" : 23,
          "line_number" : "4.4.153",
          "speaker" : "PERDITA",
          "text_entry" : "Not like a corse; or if, not to be buried,"
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "103117",
        "_score" : 14.782743,
        "_source" : {
          "type" : "line",
          "line_id" : 103118,
          "play_name" : "Twelfth Night",
          "speech_number" : 53,
          "line_number" : "1.3.95",
          "speaker" : "SIR ANDREW",
          "text_entry" : "will not be seen; or if she be, its four to one"
        }
      }
    ]
  }
}
...
```

全部清单-文字查询，请参阅[满的-文本查询]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/full-text/index)。

### 示例：确切的术语搜索

如果您想搜索一个确切的术语，例如`speaker` 字段，不需要通过相关得分对结果进行排序-级别查询效率更高：

```json
GET shakespeare/_search
{
  "query": {
    "term": {
      "speaker": "HAMLET"
    }
  }
}
```

响应包含文档匹配：

```json
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1582,
      "relation" : "eq"
    },
    "max_score" : 4.2540946,
    "hits" : [
      {
        "_index" : "shakespeare",
        "_id" : "32700",
        "_score" : 4.2540946,
        "_source" : {
          "type" : "line",
          "line_id" : 32701,
          "play_name" : "Hamlet",
          "speech_number" : 9,
          "line_number" : "1.2.66",
          "speaker" : "HAMLET",
          "text_entry" : "[Aside]  A little more than kin, and less than kind."
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "32702",
        "_score" : 4.2540946,
        "_source" : {
          "type" : "line",
          "line_id" : 32703,
          "play_name" : "Hamlet",
          "speech_number" : 11,
          "line_number" : "1.2.68",
          "speaker" : "HAMLET",
          "text_entry" : "Not so, my lord; I am too much i' the sun."
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "32709",
        "_score" : 4.2540946,
        "_source" : {
          "type" : "line",
          "line_id" : 32710,
          "play_name" : "Hamlet",
          "speech_number" : 13,
          "line_number" : "1.2.75",
          "speaker" : "HAMLET",
          "text_entry" : "Ay, madam, it is common."
        }
      }
    ]
  }
}
...
```

期限-级别查询可提供确切的匹配。因此，如果您搜索“小村庄”，您将不会收到任何匹配，因为“ hamlet”是一个关键字字段，并且在字面上存储在opensearch中，而不是以分析的形式存储。
还从字面上搜索了搜索查询“小村庄”。因此，要获得此字段的匹配，我们需要输入完全相同的字符。


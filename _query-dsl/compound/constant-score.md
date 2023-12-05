---
layout: default
title: 恒定得分
parent: 复合查询
grand_parent: 查询DSL
nav_order: 40
redirect_from:
  - /query-dsl/query-dsl/compound/constant-score/
---

# 恒定得分查询

如果您需要返回包含某个单词的文档，无论单词出现多少次，都可以使用`constant_score` 询问。A`constant_score` 查询包装过滤器查询，并在结果中分配所有文档，相关得分等于`boost` 范围。因此，所有返回的文档都具有相等的相关性评分，并且未考虑术语频率/逆文档频率（TF/IDF）。过滤器查询不计算相关性得分。此外，OpenSearch Caches经常使用过滤器查询以提高性能。

## 例子

使用以下查询返回包含单词的文档"Hamlet" 在里面`shakespeare` 指数：

```json
GET shakespeare/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "match": {
          "text_entry": "Hamlet"
        }
      },
      "boost": 1.2
    }
  }
}
```
{％包含副本-curl.html％}

结果中的所有文档分配了1.2的相关得分：

<详细信息打开降价="block">
  <summary>
    回复
  </summary>
  {： 。文本-delta}

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
      "value": 96,
      "relation": "eq"
    },
    "max_score": 1.2,
    "hits": [
      {
        "_index": "shakespeare",
        "_id": "32535",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32536,
          "play_name": "Hamlet",
          "speech_number": 48,
          "line_number": "1.1.97",
          "speaker": "HORATIO",
          "text_entry": "Dared to the combat; in which our valiant Hamlet--"
        }
      },
      {
        "_index": "shakespeare",
        "_id": "32546",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32547,
          "play_name": "Hamlet",
          "speech_number": 48,
          "line_number": "1.1.108",
          "speaker": "HORATIO",
          "text_entry": "His fell to Hamlet. Now, sir, young Fortinbras,"
        }
      },
      {
        "_index": "shakespeare",
        "_id": "32625",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32626,
          "play_name": "Hamlet",
          "speech_number": 59,
          "line_number": "1.1.184",
          "speaker": "HORATIO",
          "text_entry": "Unto young Hamlet; for, upon my life,"
        }
      },
      {
        "_index": "shakespeare",
        "_id": "32633",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32634,
          "play_name": "Hamlet",
          "speech_number": 60,
          "line_number": "",
          "speaker": "MARCELLUS",
          "text_entry": "Enter KING CLAUDIUS, QUEEN GERTRUDE, HAMLET,  POLONIUS, LAERTES, VOLTIMAND, CORNELIUS, Lords, and Attendants"
        }
      },
      {
        "_index": "shakespeare",
        "_id": "32634",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32635,
          "play_name": "Hamlet",
          "speech_number": 1,
          "line_number": "1.2.1",
          "speaker": "KING CLAUDIUS",
          "text_entry": "Though yet of Hamlet our dear brothers death"
        }
      },
      {
        "_index": "shakespeare",
        "_id": "32699",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32700,
          "play_name": "Hamlet",
          "speech_number": 8,
          "line_number": "1.2.65",
          "speaker": "KING CLAUDIUS",
          "text_entry": "But now, my cousin Hamlet, and my son,--"
        }
      },
      {
        "_index": "shakespeare",
        "_id": "32703",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32704,
          "play_name": "Hamlet",
          "speech_number": 12,
          "line_number": "1.2.69",
          "speaker": "QUEEN GERTRUDE",
          "text_entry": "Good Hamlet, cast thy nighted colour off,"
        }
      },
      {
        "_index": "shakespeare",
        "_id": "32723",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32724,
          "play_name": "Hamlet",
          "speech_number": 16,
          "line_number": "1.2.89",
          "speaker": "KING CLAUDIUS",
          "text_entry": "Tis sweet and commendable in your nature, Hamlet,"
        }
      },
      {
        "_index": "shakespeare",
        "_id": "32754",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32755,
          "play_name": "Hamlet",
          "speech_number": 17,
          "line_number": "1.2.120",
          "speaker": "QUEEN GERTRUDE",
          "text_entry": "Let not thy mother lose her prayers, Hamlet:"
        }
      },
      {
        "_index": "shakespeare",
        "_id": "32759",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32760,
          "play_name": "Hamlet",
          "speech_number": 19,
          "line_number": "1.2.125",
          "speaker": "KING CLAUDIUS",
          "text_entry": "This gentle and unforced accord of Hamlet"
        }
      }
    ]
  }
}
```
</delect>

## 参数

下表列出了所有顶部-支持的级别参数`constant_score `查询。

范围| 描述
：--- | ：---
`filter` | 文档必须匹配的过滤器查询必须在结果中返回。必需的。
`boost` | 浮动-分配为所有返回文档的相关得分的点值。选修的。默认值为1.0


---
layout: default
title: 自动完成
parent: 搜索数据
nav_order: 24
redirect_from:
  - /opensearch/search/autocomplete/
---

# 自动完成功能

自动完成在用户输入时向用户显示建议。

例如，如果用户类型"pop," OpenSearch提供了类似的建议"popcorn" 或者"popsicles." 这些建议可以使您的用户有意图，并将他们更快地进入可能的搜索词。

OpenSearch允许您设计使用每次击键更新的自动完成，提供一些相关的建议并容忍错别字。

使用以下方法之一实现自动完成：

- [前缀匹配](#prefix-matching)
- [边缘n-克匹配](#edge-n-gram-matching)
- [键入时搜索](#search-as-you-type)
- [完成建议者](#completion-suggester)

尽管前缀匹配发生在查询时间，但其他三种方法发生在索引时间。所有方法在以下各节中描述。

## 前缀匹配

前缀匹配找到与查询字符串中最后一个学期匹配的文档。

例如，假设用户将“ qui”键入搜索UI。要自动完成此短语，请使用`match_phrase_prefix` 查询搜索所有`text_entry` 以前缀开头的现场值"qui"：

```json
GET shakespeare/_search
{
  "query": {
    "match_phrase_prefix": {
      "text_entry": {
        "query": "qui",
        "slop": 3
      }
    }
  }
}
```

要使单词顺序和相对位置灵活，请指定`slop` 价值。了解`slop` 选项，请参阅[坡]({{site.url}}{{site.baseurl}}/query-dsl/full-text/match-phrase#slop)。

前缀匹配不需要任何特殊的映射。它可以与您的数据一起使用。
但是，这是一个相当多的资源-密集操作。一个前缀`a` 可以匹配数十万个任期，并且对您的用户没有用。
为了限制前缀扩展的影响，设置`max_expansions` 一个合理的数字：

```json
GET shakespeare/_search
{
  "query": {
    "match_phrase_prefix": {
      "text_entry": {
        "query": "qui",
        "slop": 3,
        "max_expansions": 10
      }
    }
  }
}
```

查询可以扩展的最大术语数量。查询“扩展”搜索术语到在指定距离内的许多匹配项中`fuzziness`。

易于实施查询-时间自动完成是以绩效为代价的。
大规模实施此功能时，我们建议使用索引-时间解决方案。带有索引-时间解决方案，您可能会遇到较慢的索引，但这是您仅支付一次的价格，而不是每个查询的价格。边缘n-克，搜索-作为-你-类型和完成建议方法是指数-时间解决方案。

## 边缘n-克匹配

在索引期间，边缘n-克将单词分为一系列n个字符，以支持更快的部分搜索词查找。

如果你n-抓取单词"quick," 结果取决于n的值。

n| 类型| n-公克
：--- | ：--- | ：---
1| UMIGRAM| [`q`，，，，`u`，，，，`i`，，，，`c`，，，，`k` 这是给出的
2| Bigram| [`qu`，，，，`ui`，，，，`ic`，，，，`ck` 这是给出的
3| Trigram| [`qui`，，，，`uic`，，，，`ick` 这是给出的
4| 四个-公克| [`quic`，，，，`uick` 这是给出的
5| 五-公克| [`quick` 这是给出的

自动完成只需要开始n-搜索短语的克，因此OpenSearch使用一种特殊类型的N-克称为 *边缘n-公克*。

边缘n-语法"quick" 导致以下结果：

- `q`
- `qu`
- `qui`
- `quic`
- `quick`

这遵循用户类型的相同序列。

配置字段以使用边缘n-克，创建一个使用一个自动完成分析仪`edge_ngram` 筛选：


```json
PUT shakespeare
{
  "mappings": {
    "properties": {
      "text_entry": {
        "type": "text",
        "analyzer": "autocomplete"
      }
    }
  },
  "settings": {
    "analysis": {
      "filter": {
        "edge_ngram_filter": {
          "type": "edge_ngram",
          "min_gram": 1,
          "max_gram": 20
        }
      },
      "analyzer": {
        "autocomplete": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "edge_ngram_filter"
          ]
        }
      }
    }
  }
}
```

此示例创建索引并实例化边缘n-克过滤器和分析仪。

这`edge_ngram_filter` 产生边缘n-克，最小n-克长度为1（单个字母），最大长度为20。

这`autocomplete` 分析仪将字符串对单个术语表示，将术语降低，然后产生边缘n-每个学期的克`edge_ngram_filter`。

使用`analyze` 测试此分析仪的操作：

```json
POST shakespeare/_analyze
{
  "analyzer": "autocomplete",
  "text": "quick"
}
```

它返回边缘n-rAMS作为令牌：

*`q`
*`qu`
*`qui`
*`quic`
*`quick`

使用`standard` 分析仪在搜索时间。否则，搜索查询将分裂为边缘n-克，您将获得与所有匹配的结果`q`，，，，`u`， 和`i`。
这是您在索引时和查询时使用不同分析仪的少数情况之一：

```json
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": {
        "query": "qui",
        "analyzer": "standard"
      }
    }
  }
}
```

响应包含匹配文档：

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
      "value": 533,
      "relation": "eq"
    },
    "max_score": 9.712725,
    "hits": [
      {
        "_index": "shakespeare",
        "_id": "22006",
        "_score": 9.712725,
        "_source": {
          "type": "line",
          "line_id": 22007,
          "play_name": "Antony and Cleopatra",
          "speech_number": 12,
          "line_number": "5.2.44",
          "speaker": "CLEOPATRA",
          "text_entry": "Quick, quick, good hands."
        }
      },
      {
        "_index": "shakespeare",
        "_id": "54665",
        "_score": 9.712725,
        "_source": {
          "type": "line",
          "line_id": 54666,
          "play_name": "Loves Labours Lost",
          "speech_number": 21,
          "line_number": "5.1.52",
          "speaker": "HOLOFERNES",
          "text_entry": "Quis, quis, thou consonant?"
        }
      }
      ...
    ]
  }
}
```

或者，指定`search_analyzer` 在映射本身中：

```json
"mappings": {
  "properties": {
    "text_entry": {
      "type": "text",
      "analyzer": "autocomplete",
      "search_analyzer": "standard"
    }
  }
}
```

## 完成建议

完成建议者接受建议清单，并将其构建为有限的-状态传感器（FST），一种基本上是图形的优化数据结构。该数据结构生活在内存中，并针对快速前缀查找进行了优化。要了解有关FST的更多信息，请参阅[维基百科](https://en.wikipedia.org/wiki/Finite-state_transducer)。

当用户类型时，完成建议将沿匹配路径一次通过FST GRAPH一个字符移动。用户输入用完之后，它将检查其余的结尾，以产生建议列表。

完成建议使您的自动完成解决方案尽可能高效，并让您明确控制其建议。

使用称为专用的字段类型[`completion`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/completion)，存储FST-喜欢索引中的数据结构：

```json
PUT shakespeare
{
  "mappings": {
    "properties": {
      "text_entry": {
        "type": "completion"
      }
    }
  }
}
```

要获得建议，请使用`search` 端点带`suggest` 范围：

```json
GET shakespeare/_search
{
  "suggest": {
    "autocomplete": {
      "prefix": "To be",
      "completion": {
        "field": "text_entry"
      }
    }
  }
}
```

词组"to be" 前缀与FST的FST匹配`text_entry` 场地：

```json
{
  "took" : 29,
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
  },
  "suggest" : {
    "autocomplete" : [
      {
        "text" : "To be",
        "offset" : 0,
        "length" : 5,
        "options" : [
          {
            "text" : "To be a comrade with the wolf and owl,--",
            "_index" : "shakespeare",
            "_id" : "50652",
            "_score" : 1.0,
            "_source" : {
              "type" : "line",
              "line_id" : 50653,
              "play_name" : "King Lear",
              "speech_number" : 68,
              "line_number" : "2.4.230",
              "speaker" : "KING LEAR",
              "text_entry" : "To be a comrade with the wolf and owl,--"
            }
          },
          {
            "text" : "To be a make-peace shall become my age:",
            "_index" : "shakespeare",
            "_id" : "78566",
            "_score" : 1.0,
            "_source" : {
              "type" : "line",
              "line_id" : 78567,
              "play_name" : "Richard II",
              "speech_number" : 20,
              "line_number" : "1.1.160",
              "speaker" : "JOHN OF GAUNT",
              "text_entry" : "To be a make-peace shall become my age:"
            }
          },
          {
            "text" : "To be a party in this injury.",
            "_index" : "shakespeare",
            "_id" : "75259",
            "_score" : 1.0,
            "_source" : {
              "type" : "line",
              "line_id" : 75260,
              "play_name" : "Othello",
              "speech_number" : 57,
              "line_number" : "5.1.93",
              "speaker" : "IAGO",
              "text_entry" : "To be a party in this injury."
            }
          },
          {
            "text" : "To be a preparation gainst the Polack;",
            "_index" : "shakespeare",
            "_id" : "33591",
            "_score" : 1.0,
            "_source" : {
              "type" : "line",
              "line_id" : 33592,
              "play_name" : "Hamlet",
              "speech_number" : 17,
              "line_number" : "2.2.67",
              "speaker" : "VOLTIMAND",
              "text_entry" : "To be a preparation gainst the Polack;"
            }
          },
          {
            "text" : "To be a public spectacle to all:",
            "_index" : "shakespeare",
            "_id" : "3709",
            "_score" : 1.0,
            "_source" : {
              "type" : "line",
              "line_id" : 3710,
              "play_name" : "Henry VI Part 1",
              "speech_number" : 6,
              "line_number" : "1.4.41",
              "speaker" : "TALBOT",
              "text_entry" : "To be a public spectacle to all:"
            }
          }
        ]
      }
    ]
  }
}
```

要指定要返回的建议数量，请使用`size` 范围：

```json
GET shakespeare/_search
{
  "suggest": {
    "autocomplete": {
      "prefix": "To n",
      "completion": {
        "field": "text_entry",
        "size": 3
      }
    }
  }
}
```

最多返回了三个文件：

```json
{
  "took" : 4109,
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
  },
  "suggest" : {
    "autocomplete" : [
      {
        "text" : "To n",
        "offset" : 0,
        "length" : 4,
        "options" : [
          {
            "text" : "To NESTOR",
            "_index" : "shakespeare",
            "_id" : "99707",
            "_score" : 1.0,
            "_source" : {
              "type" : "line",
              "line_id" : 99708,
              "play_name" : "Troilus and Cressida",
              "speech_number" : 3,
              "line_number" : "",
              "speaker" : "ULYSSES",
              "text_entry" : "To NESTOR"
            }
          },
          {
            "text" : "To name the bigger light, and how the less,",
            "_index" : "shakespeare",
            "_id" : "91884",
            "_score" : 1.0,
            "_source" : {
              "type" : "line",
              "line_id" : 91885,
              "play_name" : "The Tempest",
              "speech_number" : 91,
              "line_number" : "1.2.394",
              "speaker" : "CALIBAN",
              "text_entry" : "To name the bigger light, and how the less,"
            }
          },
          {
            "text" : "To nature none more bound; his training such,",
            "_index" : "shakespeare",
            "_id" : "40510",
            "_score" : 1.0,
            "_source" : {
              "type" : "line",
              "line_id" : 40511,
              "play_name" : "Henry VIII",
              "speech_number" : 18,
              "line_number" : "1.2.126",
              "speaker" : "KING HENRY VIII",
              "text_entry" : "To nature none more bound; his training such,"
            }
          }
        ]
      }
    ]
  }
}
```

这`suggest` 参数仅使用前缀匹配找到建议。
例如，文档"To be, or not to be" 不是结果的一部分。如果您希望将特定的文件作为建议返回，则可以手动添加精心策划的建议并添加权重以确定建议的优先级。

索引带有输入建议的文档并分配权重：

```json
PUT shakespeare/_doc/1?refresh=true
{
  "text_entry": {
    "input": [
      "To n", "To be, or not to be: that is the question:"
    ],
    "weight": 10
  }
}
```

执行相同的搜索：

```json
GET shakespeare/_search
{
  "suggest": {
    "autocomplete": {
      "prefix": "To n",
      "completion": {
        "field": "text_entry",
        "size": 3
      }
    }
  }
}
```

您将索引文档视为第一个结果：

```json
{
  "took" : 1,
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
  },
  "suggest" : {
    "autocomplete" : [
      {
        "text" : "To n",
        "offset" : 0,
        "length" : 4,
        "options" : [
          {
            "text" : "To n",
            "_index" : "shakespeare",
            "_id" : "1",
            "_score" : 10.0,
            "_source" : {
              "text_entry" : {
                "input" : [
                  "To n",
                  "To be, or not to be: that is the question:"
                ],
                "weight" : 10
              }
            }
          },
          {
            "text" : "To NESTOR",
            "_index" : "shakespeare",
            "_id" : "99707",
            "_score" : 1.0,
            "_source" : {
              "type" : "line",
              "line_id" : 99708,
              "play_name" : "Troilus and Cressida",
              "speech_number" : 3,
              "line_number" : "",
              "speaker" : "ULYSSES",
              "text_entry" : "To NESTOR"
            }
          },
          {
            "text" : "To name the bigger light, and how the less,",
            "_index" : "shakespeare",
            "_id" : "91884",
            "_score" : 1.0,
            "_source" : {
              "type" : "line",
              "line_id" : 91885,
              "play_name" : "The Tempest",
              "speech_number" : 91,
              "line_number" : "1.2.394",
              "speaker" : "CALIBAN",
              "text_entry" : "To name the bigger light, and how the less,"
            }
          }
        ]
      }
    ]
  }
}
```

您还可以通过指定查询中的拼写错误`fuzzy` 范围：

```json
GET shakespeare/_search
{
  "suggest": {
    "autocomplete": {
      "prefix": "rosenkrantz",
      "completion": {
        "field": "text_entry",
        "size": 3,
        "fuzzy" : {
            "fuzziness" : "AUTO"
        }
      }
    }
  }
}
```

结果与正确的拼写匹配：

```json
{
  "took" : 1,
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
  },
  "suggest" : {
    "autocomplete" : [
      {
        "text" : "rosenkrantz",
        "offset" : 0,
        "length" : 11,
        "options" : [
          {
            "text" : "ROSENCRANTZ:",
            "_index" : "shakespeare",
            "_id" : "35196",
            "_score" : 5.0,
            "_source" : {
              "type" : "line",
              "line_id" : 35197,
              "play_name" : "Hamlet",
              "speech_number" : 2,
              "line_number" : "4.2.1",
              "speaker" : "HAMLET",
              "text_entry" : "ROSENCRANTZ:"
            }
          }
        ]
      }
    ]
  }
}
```

您可以使用正则表达式来定义完成建议的前缀查询：

```json
GET shakespeare/_search
{
  "suggest": {
    "autocomplete": {
      "prefix": "rosen*",
      "completion": {
        "field": "text_entry",
        "size": 3
      }
    }
  }
}
```

有关更多信息，请参阅[`completion` 现场类型文档]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/completion)。

## 键入时搜索

OpenSearch有一个专用[`search_as_you_type`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/search-as-you-type) 用于搜索的现场类型-作为-你-键入功能，可以使用前缀和infix完成术语匹配。这`search_as_you_type` 字段不需要您事先设置自定义分析仪或索引建议。

首先，将字段映射为`search_as_you_type`：

```json
PUT shakespeare
{
  "mappings": {
    "properties": {
      "text_entry": {
        "type": "search_as_you_type"
      }
    }
  }
}
```

索引文档后，OpenSearch自动创建并存储其n-克和边缘n-克。例如，考虑字符串`that is the question`。首先，使用标准分析仪分为术语，将术语存储在`text_entry` 场地：

```json
[
    "that",
    "is",
    "the",
    "question"
]
```

除了存储这些术语，以下2-该领域的克将其存储在该领域`text_entry._2gram`：

```json
[
    "that is",
    "is the",
    "the question"
]
```

以下3-该领域的克将其存储在该领域`text_entry._3gram`：

```json
[
    "that is the",
    "is the question"
]
```

最后，在边缘n之后-应用了克令牌过滤器，结果术语存储在`text_entry._index_prefix` 场地：

```json
[
    "t", 
    "th", 
    "tha", 
    "that", 
    ...
]
```

然后，您可以使用任何顺序匹配术语`bool_prefix` 类型`multi-match` 询问：

```json
GET shakespeare/_search
{
  "query": {
    "multi_match": {
      "query": "uncle what",
      "type": "bool_prefix",
      "fields": [
        "text_entry",
        "text_entry._2gram",
        "text_entry._3gram"
      ]
    }
  },
  "size": 3
}
```

在结果中，单词以与查询相同的顺序显示的文档排名更高：

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4759,
      "relation" : "eq"
    },
    "max_score" : 10.437667,
    "hits" : [
      {
        "_index" : "shakespeare",
        "_id" : "2817",
        "_score" : 10.437667,
        "_source" : {
          "type" : "line",
          "line_id" : 2818,
          "play_name" : "Henry IV",
          "speech_number" : 5,
          "line_number" : "5.2.31",
          "speaker" : "HOTSPUR",
          "text_entry" : "Uncle, what news?"
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "37085",
        "_score" : 9.437667,
        "_source" : {
          "type" : "line",
          "line_id" : 37086,
          "play_name" : "Henry V",
          "speech_number" : 26,
          "line_number" : "1.2.262",
          "speaker" : "KING HENRY V",
          "text_entry" : "What treasure, uncle?"
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "79274",
        "_score" : 9.358302,
        "_source" : {
          "type" : "line",
          "line_id" : 79275,
          "play_name" : "Richard II",
          "speech_number" : 29,
          "line_number" : "2.1.187",
          "speaker" : "KING RICHARD II",
          "text_entry" : "Why, uncle, whats the matter?"
        }
      }
    ]
  }
}
```

要按顺序匹配术语，您可以使用`match_phrase_prefix` 询问：

```json
GET shakespeare/_search
{
  "query": {
    "match_phrase_prefix": {
      "text_entry": "uncle wha"
    }
  },
  "size": 3
}
```

响应包含与前缀相匹配的文档：

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 6,
      "relation" : "eq"
    },
    "max_score" : 16.37664,
    "hits" : [
      {
        "_index" : "shakespeare",
        "_id" : "2817",
        "_score" : 16.37664,
        "_source" : {
          "type" : "line",
          "line_id" : 2818,
          "play_name" : "Henry IV",
          "speech_number" : 5,
          "line_number" : "5.2.31",
          "speaker" : "HOTSPUR",
          "text_entry" : "Uncle, what news?"
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "6789",
        "_score" : 16.37664,
        "_source" : {
          "type" : "line",
          "line_id" : 6790,
          "play_name" : "Henry VI Part 2",
          "speech_number" : 60,
          "line_number" : "1.3.202",
          "speaker" : "KING HENRY VI",
          "text_entry" : "Uncle, what shall we say to this in law?"
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "7877",
        "_score" : 16.37664,
        "_source" : {
          "type" : "line",
          "line_id" : 7878,
          "play_name" : "Henry VI Part 2",
          "speech_number" : 13,
          "line_number" : "3.2.28",
          "speaker" : "KING HENRY VI",
          "text_entry" : "Where is our uncle? whats the matter, Suffolk?"
        }
      }
    ]
  }
}
```

最后，要完全匹配上一个学期，而不是作为前缀，您可以使用`match_phrase` 询问：

```json
GET shakespeare/_search
{
  "query": {
    "match_phrase": {
      "text_entry": "uncle what"
    }
  },
  "size": 5
}
```

响应包含确切的匹配：

```json
{
  "took" : 1,
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
    "max_score" : 14.437452,
    "hits" : [
      {
        "_index" : "shakespeare",
        "_id" : "2817",
        "_score" : 14.437452,
        "_source" : {
          "type" : "line",
          "line_id" : 2818,
          "play_name" : "Henry IV",
          "speech_number" : 5,
          "line_number" : "5.2.31",
          "speaker" : "HOTSPUR",
          "text_entry" : "Uncle, what news?"
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "6789",
        "_score" : 9.461917,
        "_source" : {
          "type" : "line",
          "line_id" : 6790,
          "play_name" : "Henry VI Part 2",
          "speech_number" : 60,
          "line_number" : "1.3.202",
          "speaker" : "KING HENRY VI",
          "text_entry" : "Uncle, what shall we say to this in law?"
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "100955",
        "_score" : 8.947967,
        "_source" : {
          "type" : "line",
          "line_id" : 100956,
          "play_name" : "Troilus and Cressida",
          "speech_number" : 28,
          "line_number" : "3.2.98",
          "speaker" : "CRESSIDA",
          "text_entry" : "Well, uncle, what folly I commit, I dedicate to you."
        }
      }
    ]
  }
}
```

如果您在上一个中修改文本`match_phrase` 查询并省略了最后一封信，上一个响应中的任何文件都没有返回：

```json
GET shakespeare/_search
{
  "query": {
    "match_phrase": {
      "text_entry": "uncle wha"
    }
  }
}
```

结果是空的：

```json
{
  "took" : 1,
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

有关更多信息，请参阅[`search_as_you_type` 现场类型文档]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/search-as-you-type)


---
layout: default
title: 突出显示查询匹配
parent: 搜索数据
nav_order: 23
redirect_from:
  - /opensearch/search/highlight/
---

# 突出显示查询匹配

突出显示结果中强调搜索词，因此您可以强调查询匹配项。

要突出搜索词，请添加一个`highlight` 查询块之外的参数：

```json
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": "life"
    }
  },
  "size": 3,
  "highlight": {
    "fields": {
      "text_entry": {}
    }
  }
}
```

结果中的每个文档都包含一个`highlight` 显示您的搜索词包裹在`em` 标签：

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
      "value" : 805,
      "relation" : "eq"
    },
    "max_score" : 7.450247,
    "hits" : [
      {
        "_index" : "shakespeare",
        "_id" : "33765",
        "_score" : 7.450247,
        "_source" : {
          "type" : "line",
          "line_id" : 33766,
          "play_name" : "Hamlet",
          "speech_number" : 60,
          "line_number" : "2.2.233",
          "speaker" : "HAMLET",
          "text_entry" : "my life, except my life."
        },
        "highlight" : {
          "text_entry" : [
            "my <em>life</em>, except my <em>life</em>."
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "51877",
        "_score" : 6.873042,
        "_source" : {
          "type" : "line",
          "line_id" : 51878,
          "play_name" : "King Lear",
          "speech_number" : 18,
          "line_number" : "4.6.52",
          "speaker" : "EDGAR",
          "text_entry" : "The treasury of life, when life itself"
        },
        "highlight" : {
          "text_entry" : [
            "The treasury of <em>life</em>, when <em>life</em> itself"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "39245",
        "_score" : 6.6167283,
        "_source" : {
          "type" : "line",
          "line_id" : 39246,
          "play_name" : "Henry V",
          "speech_number" : 7,
          "line_number" : "4.7.31",
          "speaker" : "FLUELLEN",
          "text_entry" : "mark Alexanders life well, Harry of Monmouths life"
        },
        "highlight" : {
          "text_entry" : [
            "mark Alexanders <em>life</em> well, Harry of Monmouths <em>life</em>"
          ]
        }
      }
    ]
  }
}
```

亮点功能可在实际字段内容上起作用。OpenSearch从存储的字段中检索这些内容（将映射设置为`true`）或来自`_source` 字段如果未存储字段。您可以从`_source` 通过设置字段`force_source` 参数为`true`。

这`highlight` 参数即使使用同义词或用于搜索本身的同义词时，也会突出显示原始术语。
{: .note}

## 获取偏移的方法

为了突出搜索词，荧光笔需要每个学期的开始和最终字符偏移。偏移标记了原始文本中术语的位置。荧光笔可以从以下来源获得偏移：

- **帖子**：当索引文档时，OpenSearch会创建一个倒置的搜索索引＆mdash;用于搜索文档的核心数据结构。发布表示倒置搜索索引，并将每个分析术语的映射存储到发生在其发生的文档列表中。如果您设置`index_options` 参数为`offsets` 映射时[文本域]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/text)，OpenSearch将每个学期的开始和最终字符偏移添加到倒置索引中。在突出显示的过程中，荧光笔直接在帖子上重新运行原始查询以找到每个学期。因此，存储偏移可以使大型字段更有效地突出显示，因为它不需要重新分析文本。存储术语偏移需要额外的磁盘空间，但使用磁盘空间少于存储术语向量。

- [**术语向量**]( If you set the [`term_vector` parameter]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/text#term-vector-parameter) 到`with_positions_offsets` 映射文本字段时，荧光笔使用`term_vector` 突出显示领域。存储术语向量需要最多的磁盘空间。但是，它使大于1 MB的田地的突出显示速度更快，多数-诸如前缀或通配符之类的术语查询，因为术语向量为每个文档提供了对条款字典的访问权限。

- **文本重新分析**：在没有帖子和术语向量的情况下，荧光笔重新分析文本以突出显示。对于每个需要突出显示的文档和每个字段，荧光笔都会在-内存索引并通过Lucene的查询执行计划重新运行原始查询以访问低-当前文档的级别匹配信息。在大多数用例中，重新分析文本效果很好。但是，对于大型田地，此方法是更多的内存和时间密集型。

## 荧光笔类型

OpenSearch支持三个荧光笔实现：`plain`，`unified`， 和`fvh` （快速矢量荧光笔）。

下表列出了为每个荧光笔获得偏移的方法。

荧光笔| 获取偏移的方法
:--- | :---
[`unified`](#the-unified-highlighter) | 术语向量如果`term_vector` 被设定为`with_positions_offsets`，<br>发布`index_options` 被设定为`offsets`，<br>文本重新分析否则。
[`fvh`](#the-fvh-highlighter) | 术语向量。
[`plain`](#the-plain-highlighter) | 文本重新分析。

### 设置荧光笔类型

要设置荧光笔类型，请在`type` 场地：

```json
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": "life"
    }
  },
  "highlight": {
    "fields": {
      "text_entry": { "type": "plain"}
    }
  }
}
```

### 这`unified` 荧光笔

这`unified` 荧光笔基于Lucene Unified荧光笔，是OpenSearch的默认荧光笔。它将文本分为句子，并将这些句子视为单个文档，并使用BM25算法对它们进行相似性评分。这`unified` 荧光笔支持精确的短语和多词-术语突出显示，包括模糊，前缀和正则正则。如果您正在使用复杂的查询来突出显示多个文档中的多个字段，我们建议使用`unified` 荧光笔开`postings` 或者`term_vector` 字段。

### 这`fvh` 荧光笔

这`fvh` 荧光笔基于Lucene快速矢量荧光笔。要使用此荧光笔，您需要存储具有位置偏移的术语向量，这会增加索引尺寸。这`fvh` 荧光笔可以将来自多个字段的匹配项结合到一个结果。它还可以根据其位置为匹配分配权重。因此，您可以在突出显示在术语匹配中提高短语匹配的查询时，将短语匹配上述匹配。此外，您可以配置`fvh` 荧光笔可以选择返回的文本片段的边界，您可以突出显示具有不同标签的多个单词。

### 这`plain` 荧光笔

这`plain` 荧光笔基于标准的Lucene荧光笔。它要求突出显示的字段单独存储`_source` 场地。这`plain` 荧光笔反映了查询匹配逻辑，特别是单词重要性和短语查询中的位置。它适用于大多数用例，但对于大型领域来说可能很慢，因为它必须重新分析文本要突出显示。

## 突出显示选项

下表描述了您可以在全局或字段级别指定的突出显示选项。场地-级别设置覆盖全局设置。

选项| 描述
:--- | :---
类型| 指定使用的荧光笔。有效值是`unified`，`fvh`， 和`plain`。默认为`unified`。
字段| 指定要搜索要突出显示的文本的字段。支持通配符表达式。如果您使用通配符，只有`text` 和`keyword` 田野被突出显示。例如，您可以设置`fields` 到`my_field*` 包括全部`text` 和`keyword` 以前缀开头的字段`my_field`。
force_source| 指定应从`_source` 字段而不是来自存储的字段值。默认为`false`。
require_field_match| 指定是否仅突出显示包含搜索查询匹配的字段。默认为`true`。要突出显示所有字段，请将此选项设置为`false`。
pre_tag| 指定突出显示文本的HTML启动标签作为字符串数组。
post_tags| 指定突出显示的文本的HTML终端标签作为字符串数组。
tags_schema| 如果将此选项设置为`styled`，OpenSearch使用已建立的-在标签模式中。在这个模式中，`pre_tags` 是`<em class="hlt1">`，`<em class="hlt2">`，`<em class="hlt3">`，`<em class="hlt4">`，`<em class="hlt5">`，`<em class="hlt6">`，`<em class="hlt7">`，`<em class="hlt8">`，`<em class="hlt9">`， 和`<em class="hlt10">`和`post_tags` 是`</em>`。
boundary_chars| 所有边界字符组合在字符串中。<br>默认值为`".,!? \t\n"`。
boundare_scanner| 仅适用于`unified` 和`fvh` 荧光笔。指定是否将突出显示的片段分为句子，单词或字符。有效值如下：<br>- `sentence`：在句子边界处拆分突出显示的片段，如[断路器](https://docs.oracle.com/javase/8/docs/api/java/text/BreakIterator.html)。您可以在`boundary_scanner_locale` 选项。<br>- `word`：拆分突出显示在单词边界处的片段，如[断路器](https://docs.oracle.com/javase/8/docs/api/java/text/BreakIterator.html)。您可以在`boundary_scanner_locale` 选项。<br>- `chars`：拆分突出显示的片段在列出的任何字符中`boundary_chars`。仅适用于`fvh` 荧光笔。
boundard_scanner_locale| 提供[语言环境](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html) 为了`boundary_scanner`。有效值是语言标签（例如，`"en-US"`）。默认为[locale.root](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html#ROOT)。
boundard_max_scan| 控制当`boundary_scanner` the的参数`fvh` 荧光笔设置为`chars`。默认值为20。
编码器| 指定是否应在返回之前对突出显示的片段进行编码。有效值是`default` （无编码）或`html` （首先逃脱HTML文本，然后插入突出显示标签）。例如，如果字段文本为`<h3>Hamlet</h3>` 和`encoder` 被设定为`html`，突出显示的文字是`"&lt;h3&gt;<em>Hamlet</em>&lt;&#x2F;h3&gt;"`。
碎片| 指定如何将文本分成突出显示的片段。仅适用于`plain` 荧光笔。有效值如下：<br>- `span` （默认值）：将文本分成相同大小的片段，但试图在突出显示的术语之间不划分文本。<br>- `simple`：将文本分成相同大小的片段。
fragment_offset| 指定要开始突出显示的字符偏移。有效`fvh` 仅荧光笔。
fragment_size| 突出显示的片段的大小，指定为字符数。如果`number_of_fragments` 设置为0，`fragment_size` 被忽略。默认值为100。
number_of_fragments| 返回片段的最大数量。如果`number_of_fragments` 设置为0，OpenSearch返回整个字段的突出显示内容。默认值为5。
命令| 突出显示的片段的排序顺序。放`order` 到`score` 通过相关性对碎片进行排序。每个荧光笔都有不同的算法用于计算相关性得分。默认为`none`。
righlight_query| 指定匹配的查询以外的查询以外的其他指定应突出显示。这`highlight_query` 当您使用更快的查询获取文档匹配和较慢的查询时，选项很有用（例如`rescore_query`）以完善结果。我们建议将搜索查询作为一部分`highlight_query`。
Matded_fields| 结合了来自不同字段的匹配项，突出显示一个字段。此功能的最常见用例是突出显示以不同方式分析并保存在多数的文本-字段。所有领域`matched_fields` 列表必须具有`term_vector` 字段设置为`with_positions_offsets`。组合匹配的字段是唯一的负载字段，因此设置其`store` 选项`yes`。仅适用于`fvh` 荧光笔。
no_match_size| 指定字符的数量，从字段的开头开始，如果没有匹配的片段可以突出显示，则返回。默认值为0。
phrase_limit| 所考虑的文档中匹配短语的数量。限制通过`fvh` 荧光笔避免消耗大量记忆。如果`matched_fields` 被使用，`phrase_limit` 指定每个匹配字段的短语数量。更高`phrase_limit` 导致查询时间增加和更多的内存消耗。仅适用于`fvh` 荧光笔。默认值为256。
max_analyzer_offset| 指定要通过突出显示请求分析的最大字符数。其余文本将不会处理。如果要突出显示的文本超过此偏移量，则返回一个空的亮点。要为突出显示请求分析的最大字符数量由`index.highlight.max_analyzed_offset`。达到此限制后，返回错误。设置`max_analyzer_offset` 比值低于`index.highlight.max_analyzed_offset` 避免错误。

统一的荧光笔的句子扫描仪将句子分割大于`fragment_size` 在第一个单词边界之后`fragment_size` 到达了。要返回整个句子而不分裂，请设置`fragment_size` 到0。
{: .note}

## 更改突出显示标签

设计您的应用程序代码以解析`highlight` 对象并在搜索术语中执行动作，例如改变其颜色，大胆，斜体性等等。

更改默认值`em` 标签，在`pretag` 和`posttag` 参数：

```json
GET shakespeare/_search
{
  "query": {
    "match": {
      "play_name": "Henry IV"
    }
  },
  "size": 3,
  "highlight": {
    "pre_tags": [
      "<strong>"
    ],
    "post_tags": [
      "</strong>"
    ],
    "fields": {
      "play_name": {}
    }
  }
}
```

播放名称在响应中的新标签突出显示：

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
      "value" : 3205,
      "relation" : "eq"
    },
    "max_score" : 3.548232,
    "hits" : [
      {
        "_index" : "shakespeare",
        "_id" : "0",
        "_score" : 3.548232,
        "_source" : {
          "type" : "act",
          "line_id" : 1,
          "play_name" : "Henry IV",
          "speech_number" : "",
          "line_number" : "",
          "speaker" : "",
          "text_entry" : "ACT I"
        },
        "highlight" : {
          "play_name" : [
            "<strong>Henry IV</strong>"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "1",
        "_score" : 3.548232,
        "_source" : {
          "type" : "scene",
          "line_id" : 2,
          "play_name" : "Henry IV",
          "speech_number" : "",
          "line_number" : "",
          "speaker" : "",
          "text_entry" : "SCENE I. London. The palace."
        },
        "highlight" : {
          "play_name" : [
            "<strong>Henry IV</strong>"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "2",
        "_score" : 3.548232,
        "_source" : {
          "type" : "line",
          "line_id" : 3,
          "play_name" : "Henry IV",
          "speech_number" : "",
          "line_number" : "",
          "speaker" : "",
          "text_entry" : "Enter KING HENRY, LORD JOHN OF LANCASTER, the EARL of WESTMORELAND, SIR WALTER BLUNT, and others"
        },
        "highlight" : {
          "play_name" : [
            "<strong>Henry IV</strong>"
          ]
        }
      }
    ]
  }
}
```

## 指定突出显示查询

默认情况下，OpenSearch仅考虑要突出显示的搜索查询。如果您使用快速查询来获取文档匹配和较慢的查询`rescore_query` 为了完善结果，突出显示精制结果很有用。您可以通过添加一个`highlight_query`：

```json
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": {
        "query": "thats my name"
      }
    }
  },
  "rescore": {
    "window_size": 20,
    "query": {
      "rescore_query": {
        "match_phrase": {
          "text_entry": {
            "query": "thats my name",
            "slop": 1
          }
        }
      },
      "rescore_query_weight": 5
    }
  },
  "_source": false,
  "highlight": {
    "order": "score",
    "fields": {
      "text_entry": {
        "highlight_query": {
          "bool": {
            "must": {
              "match": {
                "text_entry": {
                  "query": "thats my name"
                }
              }
            },
            "should": {
              "match_phrase": {
                "text_entry": {
                  "query": "that is my name",
                  "slop": 1,
                  "boost": 10.0
                }
              }
            },
            "minimum_should_match": 0
          }
        }
      }
    }
  }
}
```

## 结合来自不同领域的比赛以突出显示一个字段

您可以将来自不同字段的匹配结合在一起，以突出显示一个字段`fvh` 荧光笔。此功能的最常见用例是突出显示以不同方式分析并保存在多数的文本-字段。所有领域`matched_fields` 列表必须具有`term_vector` 字段设置为`with_positions_offsets`。组合匹配的字段是唯一的负载字段，因此设置其`store` 选项`yes`。

### 例子

为`shakespeare` 索引在哪里`text_entry` 用`standard` 分析仪，有一个`english` 用`english` 分析仪：

```json
PUT shakespeare
{
  "mappings" : {
    "properties" : {
      "text_entry" : {
        "type" :  "text",
        "term_vector": "with_positions_offsets",
        "fields": {
          "english": { 
            "type":     "text",
            "analyzer": "english",
            "term_vector": "with_positions_offsets"
          }
        }
      }
    }
  }
}
```

这`standard` 分析仪分裂`text_entry` 字段变成单个单词。您可以使用分析API操作来确认这一点：

```json
GET shakespeare/_analyze
{
  "text": "bragging of thine",
  "field": "text_entry"
}
```

响应包含在空白上的原始字符串拆分：

```json
{
  "tokens" : [
    {
      "token" : "bragging",
      "start_offset" : 0,
      "end_offset" : 8,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "of",
      "start_offset" : 9,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "thine",
      "start_offset" : 12,
      "end_offset" : 17,
      "type" : "<ALPHANUM>",
      "position" : 2
    }
  ]
}
```

这`english` 分析仪不仅将字符串分为单词，还可以驱使令牌并删除停止词。您可以通过使用分析API操作与`text_entry.english` 场地：

```json
GET shakespeare/_analyze
{
  "text": "bragging of thine",
  "field": "text_entry.english"
}
```

响应包含词干词：

```json
{
  "tokens" : [
    {
      "token" : "brag",
      "start_offset" : 0,
      "end_offset" : 8,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "thine",
      "start_offset" : 12,
      "end_offset" : 17,
      "type" : "<ALPHANUM>",
      "position" : 2
    }
  ]
}
```

搜索单词的所有形式`bragging`，使用以下查询：

```json
GET shakespeare/_search
{
  "query": {
    "query_string": {
      "query": "text_entry.english:bragging",
      "fields": [
        "text_entry"
      ]
    }
  },
  "highlight": {
    "order": "score",
    "fields": {
      "text_entry": {
        "matched_fields": [
          "text_entry",
          "text_entry.english"
        ],
        "type": "fvh"
      }
    }
  }
}
```

响应突出显示了单词的所有版本"bragging" 在里面`text_entry` 场地：

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
      "value" : 26,
      "relation" : "eq"
    },
    "max_score" : 10.153671,
    "hits" : [
      {
        "_index" : "shakespeare",
        "_id" : "56666",
        "_score" : 10.153671,
        "_source" : {
          "type" : "line",
          "line_id" : 56667,
          "play_name" : "macbeth",
          "speech_number" : 34,
          "line_number" : "2.3.118",
          "speaker" : "MACBETH",
          "text_entry" : "Is left this vault to brag of."
        },
        "highlight" : {
          "text_entry" : [
            "Is left this vault to <em>brag</em> of."
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "71445",
        "_score" : 9.284528,
        "_source" : {
          "type" : "line",
          "line_id" : 71446,
          "play_name" : "Much Ado about nothing",
          "speech_number" : 18,
          "line_number" : "5.1.65",
          "speaker" : "LEONATO",
          "text_entry" : "As under privilege of age to brag"
        },
        "highlight" : {
          "text_entry" : [
            "As under privilege of age to <em>brag</em>"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "86782",
        "_score" : 9.284528,
        "_source" : {
          "type" : "line",
          "line_id" : 86783,
          "play_name" : "Romeo and Juliet",
          "speech_number" : 8,
          "line_number" : "2.6.31",
          "speaker" : "JULIET",
          "text_entry" : "Brags of his substance, not of ornament:"
        },
        "highlight" : {
          "text_entry" : [
            "<em>Brags</em> of his substance, not of ornament:"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "44531",
        "_score" : 8.552448,
        "_source" : {
          "type" : "line",
          "line_id" : 44532,
          "play_name" : "King John",
          "speech_number" : 15,
          "line_number" : "3.1.124",
          "speaker" : "CONSTANCE",
          "text_entry" : "A ramping fool, to brag and stamp and swear"
        },
        "highlight" : {
          "text_entry" : [
            "A ramping fool, to <em>brag</em> and stamp and swear"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "63208",
        "_score" : 8.552448,
        "_source" : {
          "type" : "line",
          "line_id" : 63209,
          "play_name" : "Merchant of Venice",
          "speech_number" : 11,
          "line_number" : "3.4.79",
          "speaker" : "PORTIA",
          "text_entry" : "A thousand raw tricks of these bragging Jacks,"
        },
        "highlight" : {
          "text_entry" : [
            "A thousand raw tricks of these <em>bragging</em> Jacks,"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "73026",
        "_score" : 8.552448,
        "_source" : {
          "type" : "line",
          "line_id" : 73027,
          "play_name" : "Othello",
          "speech_number" : 75,
          "line_number" : "2.1.242",
          "speaker" : "IAGO",
          "text_entry" : "but for bragging and telling her fantastical lies:"
        },
        "highlight" : {
          "text_entry" : [
            "but for <em>bragging</em> and telling her fantastical lies:"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "85974",
        "_score" : 8.552448,
        "_source" : {
          "type" : "line",
          "line_id" : 85975,
          "play_name" : "Romeo and Juliet",
          "speech_number" : 20,
          "line_number" : "1.5.70",
          "speaker" : "CAPULET",
          "text_entry" : "And, to say truth, Verona brags of him"
        },
        "highlight" : {
          "text_entry" : [
            "And, to say truth, Verona <em>brags</em> of him"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "96800",
        "_score" : 8.552448,
        "_source" : {
          "type" : "line",
          "line_id" : 96801,
          "play_name" : "Titus Andronicus",
          "speech_number" : 60,
          "line_number" : "1.1.311",
          "speaker" : "SATURNINUS",
          "text_entry" : "Agree these deeds with that proud brag of thine,"
        },
        "highlight" : {
          "text_entry" : [
            "Agree these deeds with that proud <em>brag</em> of thine,"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "18189",
        "_score" : 7.9273787,
        "_source" : {
          "type" : "line",
          "line_id" : 18190,
          "play_name" : "As you like it",
          "speech_number" : 12,
          "line_number" : "5.2.30",
          "speaker" : "ROSALIND",
          "text_entry" : "and Caesars thrasonical brag of I came, saw, and"
        },
        "highlight" : {
          "text_entry" : [
            "and Caesars thrasonical <em>brag</em> of I came, saw, and"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "32054",
        "_score" : 7.9273787,
        "_source" : {
          "type" : "line",
          "line_id" : 32055,
          "play_name" : "Cymbeline",
          "speech_number" : 52,
          "line_number" : "5.5.211",
          "speaker" : "IACHIMO",
          "text_entry" : "And then a mind put int, either our brags"
        },
        "highlight" : {
          "text_entry" : [
            "And then a mind put int, either our <em>brags</em>"
          ]
        }
      }
    ]
  }
}
```

为单词的原始形式评分"bragging" 更高，您可以提升`text_entry` 场地：

```json
GET shakespeare/_search
{
  "query": {
    "query_string": {
      "query": "bragging",
      "fields": [
        "text_entry^5",
        "text_entry.english"
      ]
    }
  },
  "highlight": {
    "order": "score",
    "fields": {
      "text_entry": {
        "matched_fields": [
          "text_entry",
          "text_entry.english"
        ],
        "type": "fvh"
      }
    }
  }
}
```

响应列出了包含单词的文档"bragging" 第一的：

```json
{
  "took" : 17,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 26,
      "relation" : "eq"
    },
    "max_score" : 49.746853,
    "hits" : [
      {
        "_index" : "shakespeare",
        "_id" : "45739",
        "_score" : 49.746853,
        "_source" : {
          "type" : "line",
          "line_id" : 45740,
          "play_name" : "King John",
          "speech_number" : 10,
          "line_number" : "5.1.51",
          "speaker" : "BASTARD",
          "text_entry" : "Of bragging horror: so shall inferior eyes,"
        },
        "highlight" : {
          "text_entry" : [
            "Of <em>bragging</em> horror: so shall inferior eyes,"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "63208",
        "_score" : 47.077244,
        "_source" : {
          "type" : "line",
          "line_id" : 63209,
          "play_name" : "Merchant of Venice",
          "speech_number" : 11,
          "line_number" : "3.4.79",
          "speaker" : "PORTIA",
          "text_entry" : "A thousand raw tricks of these bragging Jacks,"
        },
        "highlight" : {
          "text_entry" : [
            "A thousand raw tricks of these <em>bragging</em> Jacks,"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "68474",
        "_score" : 47.077244,
        "_source" : {
          "type" : "line",
          "line_id" : 68475,
          "play_name" : "A Midsummer nights dream",
          "speech_number" : 101,
          "line_number" : "3.2.427",
          "speaker" : "PUCK",
          "text_entry" : "Thou coward, art thou bragging to the stars,"
        },
        "highlight" : {
          "text_entry" : [
            "Thou coward, art thou <em>bragging</em> to the stars,"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "73026",
        "_score" : 47.077244,
        "_source" : {
          "type" : "line",
          "line_id" : 73027,
          "play_name" : "Othello",
          "speech_number" : 75,
          "line_number" : "2.1.242",
          "speaker" : "IAGO",
          "text_entry" : "but for bragging and telling her fantastical lies:"
        },
        "highlight" : {
          "text_entry" : [
            "but for <em>bragging</em> and telling her fantastical lies:"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "39816",
        "_score" : 44.679565,
        "_source" : {
          "type" : "line",
          "line_id" : 39817,
          "play_name" : "Henry V",
          "speech_number" : 28,
          "line_number" : "5.2.138",
          "speaker" : "KING HENRY V",
          "text_entry" : "armour on my back, under the correction of bragging"
        },
        "highlight" : {
          "text_entry" : [
            "armour on my back, under the correction of <em>bragging</em>"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "63200",
        "_score" : 44.679565,
        "_source" : {
          "type" : "line",
          "line_id" : 63201,
          "play_name" : "Merchant of Venice",
          "speech_number" : 11,
          "line_number" : "3.4.71",
          "speaker" : "PORTIA",
          "text_entry" : "Like a fine bragging youth, and tell quaint lies,"
        },
        "highlight" : {
          "text_entry" : [
            "Like a fine <em>bragging</em> youth, and tell quaint lies,"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "56666",
        "_score" : 10.153671,
        "_source" : {
          "type" : "line",
          "line_id" : 56667,
          "play_name" : "macbeth",
          "speech_number" : 34,
          "line_number" : "2.3.118",
          "speaker" : "MACBETH",
          "text_entry" : "Is left this vault to brag of."
        },
        "highlight" : {
          "text_entry" : [
            "Is left this vault to <em>brag</em> of."
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "71445",
        "_score" : 9.284528,
        "_source" : {
          "type" : "line",
          "line_id" : 71446,
          "play_name" : "Much Ado about nothing",
          "speech_number" : 18,
          "line_number" : "5.1.65",
          "speaker" : "LEONATO",
          "text_entry" : "As under privilege of age to brag"
        },
        "highlight" : {
          "text_entry" : [
            "As under privilege of age to <em>brag</em>"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "86782",
        "_score" : 9.284528,
        "_source" : {
          "type" : "line",
          "line_id" : 86783,
          "play_name" : "Romeo and Juliet",
          "speech_number" : 8,
          "line_number" : "2.6.31",
          "speaker" : "JULIET",
          "text_entry" : "Brags of his substance, not of ornament:"
        },
        "highlight" : {
          "text_entry" : [
            "<em>Brags</em> of his substance, not of ornament:"
          ]
        }
      },
      {
        "_index" : "shakespeare",
        "_id" : "44531",
        "_score" : 8.552448,
        "_source" : {
          "type" : "line",
          "line_id" : 44532,
          "play_name" : "King John",
          "speech_number" : 15,
          "line_number" : "3.1.124",
          "speaker" : "CONSTANCE",
          "text_entry" : "A ramping fool, to brag and stamp and swear"
        },
        "highlight" : {
          "text_entry" : [
            "A ramping fool, to <em>brag</em> and stamp and swear"
          ]
        }
      }
    ]
  }
}
```

## 查询限制

注意以下限制：

- 当提取术语以突出显示时，荧光笔不会反映查询的布尔逻辑。因此，对于一些复杂的布尔查询，例如使用嵌套布尔查询和查询`minimum_should_match`，OpenSearch可能会突出显示与查询匹配不符的术语。
- 这`fvh` 荧光笔不支持跨度查询


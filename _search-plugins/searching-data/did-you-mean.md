---
layout: default
title: Did-you-mean
parent: 搜索数据
nav_order: 25
redirect_from:
  - /opensearch/search/did-you-mean/
---

# Did-you-mean

这`Did-you-mean` Suggester显示，建议对拼写错误的搜索词进行更正。

例如，如果用户类型"fliud," OpenSearch提出了一个更正的搜索词"fluid." 然后，您可以向用户建议更正的术语，甚至可以自动更正搜索词。

您可以实现`did-you-mean` 建议使用以下方法之一：

- 用一个[学期建议](#term-suggester) 建议对单个单词进行更正。
- 用一个[Shere Suggester](#phrase-suggester) 建议对短语进行更正。

## 学期建议

使用Suggester一词建议单个单词的更正拼写。
Suggester一词使用[编辑距离](https://en.wikipedia.org/wiki/Edit_distance) 计算建议。

编辑距离是单个的数量-字符插入，删除或替换需要执行一个术语以匹配另一个术语。例如，更改单词"cat" 到"hats"，你需要替代"h" 为了"c" 并插入"s"，因此在这种情况下的编辑距离为2。

要使用Suggester一词，您不需要任何特殊的字段映射来索引。默认情况下，字符串字段类型被映射为`text`。A`text` 分析了字段，因此`title` 在下面的示例中，将其化为单个单词。索引以下文档会创建一个`books` 索引在哪里`title` 是一个`text` 场地：

```json
PUT books/_doc/1
{
  "title": "Design Patterns (Object-Oriented Software)"
}

PUT books/_doc/2
{
  "title": "Software Architecture Patterns Explained"
}
```

要检查字符串如何分为令牌，您可以使用`_analyze` 端点。要应用该字段使用的相同分析仪，您可以在该字段中指定字段的名称`field` 范围：

```json
GET books/_analyze
{
  "text": "Design Patterns (Object-Oriented Software)",
  "field": "title"
}
```

默认分析仪（`standard`）在单词边界上拆分一个字符串，删除标点符号并降低令牌：

```json
{
  "tokens" : [
    {
      "token" : "design",
      "start_offset" : 0,
      "end_offset" : 6,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "patterns",
      "start_offset" : 7,
      "end_offset" : 15,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "object",
      "start_offset" : 17,
      "end_offset" : 23,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "oriented",
      "start_offset" : 24,
      "end_offset" : 32,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "software",
      "start_offset" : 33,
      "end_offset" : 41,
      "type" : "<ALPHANUM>",
      "position" : 4
    }
  ]
}
```

要获取拼写错误的搜索词的建议，请使用Suggester一词。指定需要建议的输入文本`text` 字段，并指定从中获得建议的字段`field` 场地：

```json
GET books/_search
{
  "suggest": {
    "spell-check": {
      "text": "patern",
      "term": {
        "field": "title"
      }
    }
  }
}
```

该术语建议返回了输入文本的校正列表`options` 大批：

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
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "suggest" : {
    "spell-check" : [
      {
        "text" : "patern",
        "offset" : 0,
        "length" : 6,
        "options" : [
          {
            "text" : "patterns",
            "score" : 0.6666666,
            "freq" : 2
          }
        ]
      }
    ]
  }
}
```

这`score` 值是根据编辑距离计算的。分数越高，建议越好。这`freq` 是表示该项出现在指定索引的文档中的次数的频率。

您可以在一个请求中包含一些建议。以下示例使用Sugnester一词进行两个不同的建议：

```json
GET books/_search
{
  "suggest": {
    "spell-check1" : {
      "text" : "patern",
      "term" : {
        "field" : "title"
      }
    },
    "spell-check2" : {
      "text" : "desing",
      "term" : {
        "field" : "title"
      }
    }
  }
}
```

要在多个字段中接收有关相同输入文本的建议，您可以在全球定义文本以避免重复：

```json
GET books/_search
{
  "suggest": {
    "text" : "patern",
    "spell-check1" : {
      "term" : {
        "field" : "title"
      }
    },
    "spell-check2" : {
      "term" : {
        "field" : "subject"
      }
    }
  }
}
```

如果`text` 在全球和个人建议级别上指定的建议-级别值覆盖全球值。

### 定期建议选择

您可以为Suggester一词指定以下选项。

选项| 描述
：--- | ：---
场地| 从中获取建议的字段。必需的。可以为每个建议或全球设置。
分析仪| 分析仪可以分析输入文本。默认为配置的分析仪`field`。
尺寸| 输入文本中每个令牌的最大建议数量。
种类| 指定在响应中应如何分类建议。有效值是：<br>- `score`：按相似度分数排序，然后按文档频率，然后用术语本身进行排序。<br>- `frequency`：按文档频率进行排序，然后按相似度得分，然后按术语本身进行排序。
建议_mode| 建议模式指定应在响应中包含建议的术语。有效值是：<br>- `missing`：仅针对不在索引中的输入文本项返回建议。<br>- `popular`：返回建议仅在文档中发生的建议比原始输入文本更频繁。<br>- `always`：始终返回输入文本中每个术语的建议。<br>默认值为`missing`。
max_edit| 建议的最大编辑距离。有效值在[1，2]范围内。默认值为2。
prefix_length| 指定最小长度的整数必须是开始返回建议。如果前缀`prefix_length` 不匹配，但是搜索词仍在编辑距离之内，没有返回建议。默认值为1。较高的值可以提高拼写检查的性能，因为拼写错误不会在单词开头发生。
min_word_length| 必须将最小长度提出建议才能包括在响应中。默认值为4。
shard_size| 从每个碎片获得的最大候选建议数量。毕竟考虑了所有候选建议，最高`shard_size` 提出建议。默认值等于`size` 价值。碎片-级文档频率可能不是确切的，因为术语可能存在于不同的碎片中。如果`shard_size` 大于`size`，建议的文件频率更准确，而绩效降低为代价。
max_inspections| 的乘法因子`shard_size`。候选人建议的最大数量开放搜索检查以查找建议是计算的`shard_size` 乘以`max_inspection`。可以以降低性能成本提高准确性。默认值为5。
min_doc_freq| 建议将其退还建议的最小数量或百分比。仅通过高碎片返回建议，可以提高准确性-级文档频率。有效值是表示文档频率或浮动在表示文档百分比的[0，1]范围内的整数。默认值为0（功能禁用）。
max_term_freq| 为了退还建议，应出现建议的最大文档数量。有效值是表示文档频率或浮动在表示文档百分比的[0，1]范围内的整数。默认值为0.01。不包括高-频率术语改善了拼写检查性能，因为高-频率术语通常正确拼写。使用碎片-级文档频率。
string_distance| 用于确定相似性的编辑距离算法。有效值是：<br>- `internal`：基于[达米拉（Damerau）-Levenshtein算法](https://en.wikipedia.org/wiki/Damerau%E2%80%93Levenshtein_distance) 但是，高度优化用于比较索引中术语的编辑距离。<br>- `damerau_levenshtein`：基于编辑距离算法[达米拉（Damerau）-Levenshtein算法](https://en.wikipedia.org/wiki/Damerau%E2%80%93Levenshtein_distance)。<br>- `levenshtein`：基于编辑距离算法[Levenshtein编辑距离算法](https://en.wikipedia.org/wiki/Levenshtein_distance)。<br>- `jaro_winkler`：基于编辑距离算法[贾罗-Winkler算法](https://en.wikipedia.org/wiki/Jaro%E2%80%93Winkler_distance)。<br>- `ngram`：基于字符n的编辑距离算法-克。

## Shere Suggester

实施`did-you-mean`，使用短语建议。
该短语建议类似于Suggester一词，除了使用n-克语言模型建议整个短语而不是单个单词。

要设置短语建议，请创建一个称为的自定义分析仪`trigram` 使用的是`shingle` 过滤和下尺寸令牌。该过滤器类似于`edge_ngram` 过滤器，但它适用于单词而不是字母。然后，使用您创建的自定义分析仪来配置您将要从中采购建议的字段：

```json
PUT books2
{
  "settings": {
    "index": {
      "analysis": {
        "analyzer": {
          "trigram": {
            "type": "custom",
            "tokenizer": "standard",
            "filter": [
              "lowercase",
              "shingle"
            ]
          }
        },
        "filter": {
          "shingle": {
            "type": "shingle",
            "min_shingle_size": 2,
            "max_shingle_size": 3
          }
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "trigram": {
            "type": "text",
            "analyzer": "trigram"
          }
        }
      }
    }
  }
}
```

将文档索引到新索引：

```json
PUT books2/_doc/1
{
  "title": "Design Patterns"
}

PUT books2/_doc/2
{
  "title": "Software Architecture Patterns Explained"
}
```

假设用户搜索不正确的短语：

```json
GET books2/_search
{
  "suggest": {
    "phrase-check": {
      "text": "design paterns",
      "phrase": {
        "field": "title.trigram"
      }
    }
  }
}
```

该短语建议返回校正的短语：

```json
{
  "took" : 4,
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
    "phrase-check" : [
      {
        "text" : "design paterns",
        "offset" : 0,
        "length" : 14,
        "options" : [
          {
            "text" : "design patterns",
            "score" : 0.31666178
          }
        ]
      }
    ]
  }
}
```

要突出建议，请设置[`highlight`]({{site.url}}{{site.baseurl}}/opensearch/search/highlight) “建议”的字段：

```json
GET books2/_search
{
  "suggest": {
    "phrase-check": {
      "text": "design paterns",
      "phrase": {
        "field": "title.trigram",
        "gram_size": 3,
        "highlight": {
          "pre_tag": "<em>",
          "post_tag": "</em>"
        }
      }
    }
  }
}
```

结果包含突出显示的文本：

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
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "suggest" : {
    "phrase-check" : [
      {
        "text" : "design paterns",
        "offset" : 0,
        "length" : 14,
        "options" : [
          {
            "text" : "design patterns",
            "highlighted" : "design <em>patterns</em>",
            "score" : 0.31666178
          }
        ]
      }
    ]
  }
}
```

### 短语建议选项

您可以为“建议”一词指定以下选项。

选项| 描述
：--- | ：---
场地| 用于N的字段-克查找。该短语建议使用此字段来计算建议分数。必需的。
gram_size| 最大尺寸`n` n-克（带状疱疹）中的克（带状疱疹）。如果字段不包含n-克（带状疱疹），省略此选项或将其设置为1。如果字段使用带状疱疹过滤器，并且`gram_size` 未设置，`gram_size` 被设定为`max_shingle_size`。
real_word_error_likelihood| 即使在字典中存在术语，该术语的可能性也是如此。默认值为0.95（词典中的单词的5％是拼写错误的）。
信心| 置信度是一个浮点因子，乘以输入短语的分数，以计算其他建议的阈值得分。仅返回分数高于阈值的建议。1.0的置信度水平只会返回得分高于输入短语的建议。如果`confidence` 设置为0，顶部`size` 候选人退还。默认值为1。
max_errors| 为了返回建议，该条款的最大数量或百分比可能是错误的（拼写错误）。有效的值是代表表示条款百分比（0，1）范围内的条款或浮动数的整数。默认值为1（最多只有一个拼写错误的返回建议）。将此值设置为高数字可以降低性能。我们建议设置`max_errors` 相对于查询执行所花费的时间，减少了较低的数字或2或2的时间。
分隔器| Bigram字段中术语的分离器。默认为空格字符。
尺寸| 为每个查询项生成的候选建议数量。指定更高的值可能会导致返回较高的编辑距离。默认值为5。
分析仪| 分析仪可以分析建议文本。默认为配置的分析仪`field`。
shard_size| 从每个碎片获得的最大候选建议数量。毕竟考虑了所有候选建议，最高`shard_size` 提出建议。默认值为5。
[整理](#collate-field)| 用于修剪索引中没有匹配文档的建议。
整理| 指定一个查询，该查询需要检查建议，以修剪索引中没有匹配文档的建议。
整理| 指定是否返回所有建议。如果`prune` 被设定为`false`，仅返回那些具有匹配文档的建议。如果`prune` 被设定为`true`，所有建议均已返回；每个建议都有一个`collate_match` 那是`true` 如果建议有匹配的文档，并且是`false` 否则。默认为`false`。
强调| 配置建议突出显示。两个都`pre_tag` 和`post_tag` 需要值。
亮点.pre_tag| 突出显示的起始标签。
亮点.post_tag| 突出显示的结尾标签。
[平滑](#smoothing-models) | 平滑模型，以平衡索引中存在的带状疱疹的重量与索引中存在的带状板的重量很少。


### 整理场

为了滤除不会返回任何结果的拼写检查的建议，您可以使用`collate` 场地。该字段包含一个为每个返回建议运行的脚本查询。看[搜索模板]({{site.url}}{{site.baseurl}}/opensearch/search-template) 有关构建模板查询的信息。您可以使用`{% raw %}{{suggestion}}{% endraw %}` 变量，或者您可以在`params` 字段（建议值将添加到您指定的变量中）。

有关建议的整理查询仅在提出建议的碎片上运行。需要查询。

另外，如果`prune` 参数设置为`true`， A`collate_match` 字段添加到每个建议中。如果查询不返回结果，`collate_match` 价值是`false`。然后，您可以根据`collate_match` 场地。这`prune` 参数的默认值为`false`。

例如，以下查询配置`collate` 运行一个字段`match_phrase` 查询匹配`title` 目前的建议：

```json
GET books2/_search
{
  "suggest": {
    "phrase-check": {
      "text": "design paterns",
      "phrase": {
        "field": "title.trigram",
        "collate" : {
          "query" : {
            "source": {
              "match_phrase" : {
                "title": "{{suggestion}}"
              }
            }
          },
          "prune": "true"
        }
      }
    }
  }
}
```

由此产生的建议包含`collate_match` 字段设置为`true`，这意味着`match_phrase` 查询将返回该建议的匹配文件：

```json
{
  "took" : 7,
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
    "phrase-check" : [
      {
        "text" : "design paterns",
        "offset" : 0,
        "length" : 14,
        "options" : [
          {
            "text" : "design patterns",
            "score" : 0.56759655,
            "collate_match" : true
          }
        ]
      }
    ]
  }
}
```


### 平滑模型

在大多数用例中，计算建议的分数时，您不仅要考虑木瓦的频率，而且要考虑木瓦的尺寸。平滑模型用于计算不同尺寸的带状疱疹的分数，平衡频繁和不经常的带状疱疹的重量。

支持以下平滑模型。

模型| 描述
：--- | ：---
愚蠢的_backoff| 退回到较低-订单n-克模型，如果较高-订单n-克计数为0，乘以较低-订单n-按恒定因素进行克模型（克`discount`）。这是默认平滑模型。
愚蠢的backoff.discount| 乘以较低的因素-订单n-克模型。选修的。默认值为0.4。
拉普拉斯| 使用添加平滑，添加常数`alpha` 所有计数以平衡权重。
laplace.alpha| 将常数添加到所有计数中，以平衡权重，通常为1.0或更小。选修的。默认值为0.5。

默认情况下，OpenSearch使用愚蠢的向后模型＆mdash;一种简单的算法，该算法从最高阶段的木瓦开始，并降低-订购瓦（如果更高）-找不到订单瓦。例如，如果您设置了“建议”一词为3-克，2-克和1-克，愚蠢的退缩模型首先检查了3-克。如果没有3-克，它检查2-克，但将分数乘以`discount` 因素。如果没有2-克，它检查1-克，但再次将分数乘以`discount` 因素。在大多数情况下，愚蠢的退缩模型效果很好。如果您需要选择拉普拉斯平滑模型，请在`smoothing` 范围：

```json
GET books2/_search
{
  "suggest": {
    "phrase-check": {
      "text": "design paterns",
      "phrase": {
        "field": "title.trigram",
        "size" : 1,
        "smoothing" : {
          "laplace" : {
            "alpha" : 0.7
          }
        }
      }
    }
  }
}
```

### 候选发电机

候选生成器根据输入文本中的术语提供可能的建议条款。有一个可用的候选生成器＆mdash;`direct_generator`。直接发电机的功能类似于术语Suggester：在输入文本中的每个术语也被调用。该短语建议支持多个候选生成器，其中每个发电机在输入文本中的每个项都被调用。它还可以让您指定一个PRE-过滤器（分析仪在输入文本阶段之前分析输入文本阶段）和帖子-过滤器（分析仪在返回之前分析生成的建议）。

为Suggester设置直接发电机：

```json
GET books2/_search
{
  "suggest": {
    "text": "design paterns",
    "phrase-check": {
      "phrase": {
        "field": "title.trigram",
        "size": 1,
        "direct_generator": [
          {
            "field": "title.trigram",
            "suggest_mode": "always",
            "min_word_length": 3
          }
        ]
      }
    }
  }
}
```

您可以指定以下直接生成器选项。

选项| 描述
：--- | ：---
场地| 从中获取建议的字段。必需的。可以为每个建议或全球设置。
尺寸| 输入文本中每个令牌的最大建议数量。
建议_mode| 建议模式指定应包括每个碎片上生成的建议的术语。建议模式适用于每个碎片的建议，在组合不同碎片的建议时未检查。因此，如果建议模式为`missing`，如果一个碎片中缺少该术语，但在另一个碎片上存在，将会返回建议。有效值是：<br>- `missing`：仅针对不在碎片中的输入文本项返回建议。<br>- `popular`：返回建议仅在文档中发生的建议比碎片上的原始输入文本更频繁。<br>- `always`：始终返回建议。<br>默认值为`missing`。
max_edit| 建议的最大编辑距离。有效值在[1，2]范围内。默认值为2。
prefix_length| 指定最小长度的整数必须是开始返回建议。如果前缀`prefix_length` 不匹配，但搜索词仍在编辑距离之内，没有返回建议。默认值为1。较高的值可以提高拼写检查的性能，因为拼写错误不会在单词开头发生。
min_word_length| 必须提出建议的最小长度才能包括在内。默认值为4。
max_inspections| 的乘法因子`shard_size`。候选人建议的最大数量开放搜索检查以查找建议是计算的`shard_size` 乘以`max_inspection`。可以以降低性能成本提高准确性。默认值为5。
min_doc_freq| 为了退还建议，应出现建议的最小数量或百分比。仅通过高碎片返回建议，可以提高准确性-级文档频率。有效值是表示文档频率或浮动在表示文档百分比的[0，1]范围内的整数。默认值为0（功能禁用）。
max_term_freq| 为了退还建议，应出现建议的最大文档数量。有效值是表示文档频率或浮动在表示文档百分比的[0，1]范围内的整数。默认值为0.01。不包括高-频率术语改善了拼写检查性能，因为高-频率术语通常正确拼写。使用碎片-级文档频率。
pre_filter| 在生成建议之前，将应用于生成器的每个输入文本令牌应用于生成器。
post_filter| 一个分析仪将其应用于每个生成的建议之前，然后将其传递给短语得分手。


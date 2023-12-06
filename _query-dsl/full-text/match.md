---
layout: default
title: 匹配
parent: 全文查询
grand_parent: 查询DSL
nav_order: 10
---

# 匹配查询

使用`match` 查询满足-在特定文档字段上进行文本搜索。如果您运行`match` 查询[`text`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/text/) 字段，`match` 询问[分析]({{site.url}}{{site.baseurl}}/analyzers/index/) 提供的搜索字符串并返回与字符串任何条款匹配的文档。如果您运行`match` 确切查询-值字段，它返回与确切值匹配的文档。精确搜索的首选方法-值字段是使用过滤器，因为与查询不同，过滤器被缓存。

以下示例显示了一个基本`match` 查询单词`wind` 在里面`title`：

```json
GET _search
{
  "query": {
    "match": {
      "title": "wind"
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
    "match": {
      "title": {
        "query": "wind",
        "analyzer": "stop"
      }
    }
  }
}
```
{% include copy-curl.html %}

## 例子

在以下示例中，您将使用包含以下文档的索引：

```json
PUT testindex/_doc/1
{
  "title": "Let the wind rise"
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

```json
PUT testindex/_doc/3
{
  "title": "Rise is gone"
}
```
{% include copy-curl.html %}

## 操作员

如果一个`match` 查询是在`text` 字段，用在`analyzer` 范围。然后，使用在`operator` 范围。默认操作员是`OR`，所以查询`wind rise` 已更改为`wind OR rise`。在此示例中，此查询返回文档1--3因为每个文档的术语与查询匹配。指定`and` 操作员，使用以下查询：

```json
GET testindex/_search
{
  "query": {
    "match": {
      "title": {
        "query": "wind rise",
        "operator": "and"
      }
    }
  }
}
```
{% include copy-curl.html %}

查询被构造为`wind AND rise` 并将文档1作为匹配文档返回：

<details closed markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}

```json
{
  "took": 17,
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
    "max_score": 1.2667098,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 1.2667098,
        "_source": {
          "title": "Let the wind rise"
        }
      }
    ]
  }
}
```

</delect>

### 最低应匹配

您可以通过指定文档必须匹配文档必须匹配的最小条款数[`minimum_should_match`]({{site.url}}{{site.baseurl}}/query-dsl/minimum-should-match/) 范围：

```json
GET testindex/_search
{
  "query": {
    "match": {
      "title": {
        "query": "wind rise",
        "operator": "or",
        "minimum_should_match": 2
      }
    }
  }
}
```
{% include copy-curl.html %}

现在需要文档匹配这两个条款，因此仅返回文档1（这等同于`and` 操作员）：

<details closed markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}

```json
{
  "took": 23,
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
    "max_score": 1.2667098,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 1.2667098,
        "_source": {
          "title": "Let the wind rise"
        }
      }
    ]
  }
}
```
</details>

## 分析仪

因为在此示例中，您没有明确指定分析仪，默认`standard` 使用分析仪。默认分析仪不会执行干扰，因此，如果您运行查询`the wind rises`，您没有收到结果，因为令牌`rises` 不匹配令牌`rise`。要更改搜索分析仪，请在`analyzer` 场地。例如，以下查询使用`english` 分析仪：

```json
GET testindex/_search
{
  "query": {
    "match": {
      "title": {
        "query": "the wind rises",
        "operator": "and",
        "analyzer": "english"
      }
    }
  }
}
```
{% include copy-curl.html %}

这`english` 分析仪删除了停止词`the` 并执行茎，产生令牌`wind` 和`rise`。后者令牌匹配文档1，结果在结果中返回：

<details closed markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}

```json
{
  "took": 19,
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
    "max_score": 1.2667098,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 1.2667098,
        "_source": {
          "title": "Let the wind rise"
        }
      }
    ]
  }
}
```
</details>

## 空查询

在某些情况下，分析仪可能会从查询中删除所有令牌。例如，`english` 分析仪删除停止单词，因此在查询中`and OR or`，所有令牌均已删除。要检查分析仪的行为，您可以使用[分析API]({{site.url}}{{site.baseurl}}/api-reference/analyze-apis/#apply-a-built-in-analyzer)：

```json
GET testindex/_analyze
{
  "analyzer" : "english",
  "text" : "and OR or"
}
```
{% include copy-curl.html %}

如预期的那样，查询不会产生任何令牌：

```json
{
  "tokens": []
}
```

您可以在`zero_terms_query` 范围。环境`zero_terms_query` 到`all` 返回索引中的所有文档，然后将其设置为`none` 返回没有文件：

```json
GET testindex/_search
{
  "query": {
    "match": {
      "title": {
        "query": "and OR or",
        "analyzer" : "english",
        "zero_terms_query": "all"
      }
    }
  }
}
```
{% include copy-curl.html %}

## 模糊

要考虑错别字，您可以指定`fuzziness` 对于您的查询，即以下任何一个：

- 一个指定最大允许的整数[Levenshtein距离](https://en.wikipedia.org/wiki/Levenshtein_distance) 对于此编辑。
- `AUTO`：
  - 0-2个字符的字符串必须完全匹配。
  - 3-5个字符的字符串允许1个编辑。
  - 字符串超过5个字符允许2个编辑。

环境`fuzziness` 默认`AUTO` 在大多数情况下，价值最有效：

```json
GET testindex/_search
{
  "query": {
    "match": {
      "title": {
        "query": "wnid",
        "fuzziness": "AUTO"
      }
    }
  }
}
```
{% include copy-curl.html %}

令牌`wnid` 火柴`wind` 查询返回文档1和2：

<details closed markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}

```json
{
  "took": 31,
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
    "max_score": 0.47501624,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.47501624,
        "_source": {
          "title": "Let the wind rise"
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.47501624,
        "_source": {
          "title": "Gone with the wind"
        }
      }
    ]
  }
}
```
</details>

### 前缀长度

拼写错误很少发生在单词的开头。因此，您可以指定匹配的前缀必须是返回结果中的文档的最小长度。例如，您可以将上述查询更改为包括`prefix_length`：

```json
GET testindex/_search
{
  "query": {
    "match": {
      "title": {
        "query": "wnid",
        "fuzziness": "AUTO",
        "prefix_length": 2
      }
    }
  }
}
```
{% include copy-curl.html %}

前面的查询返回没有结果。如果您更改`prefix_length` 到1，由于令牌的第一个字母，文件1和2被返回`wnid` 没有拼写错误。

### 换位

在上一个示例中，单词`wnid` 包含一个换位（`in` 被更改为`ni`）。默认情况下，模糊匹配中允许换位，但是您可以通过设置禁止它们`fuzzy_transpositions` 到`false`：

```json
GET testindex/_search
{
  "query": {
    "match": {
      "title": {
        "query": "wnid",
        "fuzziness": "AUTO",
        "fuzzy_transpositions": false
      }
    }
  }
}
```
{% include copy-curl.html %}

现在查询返回没有结果。

## 同义词

如果您使用`synonym_graph` 过滤器和`auto_generate_synonyms_phrase_query` 被设定为`true` （默认），OpenSearch将查询解析为术语，然后结合术语以生成一个[短语查询](https://lucene.apache.org/core/8_9_0/core/org/apache/lucene/search/PhraseQuery.html) 用于多-术语同义词。例如，如果指定`ba,batting average` 作为同义词和搜索`ba`，OpenSearch搜索`ba OR "batting average"`。

匹配多-带有连词的术语同义词，设置`auto_generate_synonyms_phrase_query` 到`false`：

```json
GET /testindex/_search
{
  "query": {
    "match": {
      "text": {
        "query": "good ba",
        "auto_generate_synonyms_phrase_query": false
      }
    }
  }
}
```
{% include copy-curl.html %}

产生的查询是`ba OR (batting AND average)`。

## 参数

查询接受字段的名称（`<field>`）作为顶部-级别参数：

```json
GET _search
{
  "query": {
    "match": {
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
`auto_generate_synonyms_phrase_query` | 布尔| 指定是否创建[匹配短语查询]({{site.url}}{{site.baseurl}}/query-dsl/full-text/match-phrase/) 自动用于多人-术语同义词。例如，如果指定`ba,batting average` 作为同义词和搜索`ba`，OpenSearch搜索`ba OR "batting average"` （如果此选项是`true`） 或者`ba OR (batting AND average)` （如果此选项是`false`）。默认为`true`。
`analyzer` | 细绳| 这[分析仪]({{site.url}}{{site.baseurl}}/analyzers/index/) 用于引导查询字符串文本。默认是索引-指定的时间分析仪`default_field`。如果未针对`default_field`， 这`analyzer` 是索引的默认分析仪。
`boost` | 漂浮的-观点| 通过给定的乘数增强子句。对于在复合查询中称量从句有用。[0，1）中的值降低了相关性，并且值大于1的相关性。默认为`1`。
`enable_position_increments` | 布尔| 什么时候`true`，结果查询知道位置增量。当删除停止单词留下不必要的时，此设置很有用"gap" 在术语之间。默认为`true`。
`fuzziness` | 细绳| 在确定一个术语是否匹配值时，将一个单词更改为另一个单词所需的字符编辑数量（插入，删除，替换）。例如，`wined` 和`wind` 是1.有效值是非-负整数或`AUTO`。默认，`AUTO`，根据每个学期的长度选择一个值，对于大多数用例，是一个不错的选择。
`fuzzy_rewrite` | 细绳| 确定OpenSearch如何重写查询。有效值是`constant_score`，`scoring_boolean`，`constant_score_boolean`，`top_terms_N`，`top_terms_boost_N`， 和`top_terms_blended_freqs_N`。如果是`fuzziness` 参数不是`0`，查询使用`fuzzy_rewrite` 的方法`top_terms_blended_freqs_${max_expansions}` 默认情况下。默认为`constant_score`。
`fuzzy_transpositions` | 布尔| 环境`fuzzy_transpositions` 到`true` （默认）在插入，删除和替代操作中添加相邻字符的互换`fuzziness` 选项。例如，`wind` 和`wnid` 是1`fuzzy_transpositions` 是真的（交换"n" 和"i"）和2如果是错误的（删除"n"， 插入"n"）。如果`fuzzy_transpositions` 是错误的，`rewind` 和`wnid` 距离有相同的距离（2）`wind`，尽管人类越来越多-以中心的看法`wnid` 是一个明显的错别字。对于大多数用例，默认值是一个不错的选择。
`lenient` | 布尔| 环境`lenient` 到`true` 忽略查询和文档字段之间的数据类型不匹配。例如，一个查询字符串`"8.2"` 可以匹配类型字段`float`。默认为`false`。
`max_expansions` | 正整数|  查询可以扩展的最大术语数量。模糊的查询“扩展为”在指定距离内的许多匹配术语`fuzziness`。然后OpenSearch尝试匹配这些术语。默认为`50`。
`minimum_should_match` | 正整数，正数，正百分比，组合| 如果查询字符串包含多个搜索术语，并且您使用`or` 操作员，需要将文档匹配的术语数量被视为匹配。例如，如果`minimum_should_match` 是2，`wind often rising` 不匹配`The Wind Rises.` 如果`minimum_should_match` 是`1`， 它匹配。有关详细信息，请参阅[最低应匹配]({{site.url}}{{site.baseurl}}/query-dsl/minimum-should-match/)。
`operator` | 细绳| 如果查询字符串包含多个搜索术语，则所有术语是否需要匹配（`AND`）或只需要一个术语匹配（`OR`）文档被视为匹配项。有效值是：<br>- `OR`：字符串`to be` 被解释为`to OR be`<br>- `AND`：字符串`to be` 被解释为`to AND be`<br>默认值为`OR`。
`prefix_length` | 非-负整数| 在模糊性中未考虑的领先角色的数量。默认为`0`。
`zero_terms_query` | 细绳| 在某些情况下，分析仪从查询字符串中删除了所有术语。例如，`stop` 分析仪从字符串中删除所有术语`an but this`。在这些情况下，`zero_terms_query` 指定是否匹配不匹配文档（`none`）或所有文件（`all`）。有效值是`none` 和`all`。默认为`none`


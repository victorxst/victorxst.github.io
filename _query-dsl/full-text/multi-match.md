---
layout: default
title: Multi-match
parent: 全文查询
grand_parent: 查询DSL
nav_order: 50
---

# 多-匹配查询

多-匹配操作的功能与[匹配]({{site.url}}{{site.baseurl}}/query-dsl/full-text/match/) 手术。您可以使用`multi_match` 查询搜索多个字段。

这`^` "boosts" 某些字段。提升是乘数在一个领域中比其他领域的比赛更重的乘数。在下面的示例中，匹配"wind" 在标题领域的影响`_score` 在情节领域中的比赛四倍：

```json
GET _search
{
  "query": {
    "multi_match": {
      "query": "wind",
      "fields": ["title^4", "plot"]
    }
  }
}
```
{% include copy-curl.html %}

结果是 *诸如 *风 *和 *随风 *之类的电影都接近搜索结果的顶部，而诸如 *Twister *之类的电影大概是"wind" 在他们的情节摘要中，靠近底部。

您可以在字段名称中使用通配符。例如，以下查询将搜索`speaker` 野外和所有以开始的字段`play_`， 例如，`play_name` 或者`play_title`：

```json
GET _search
{
  "query": {
    "multi_match": {
      "query": "hamlet",
      "fields": ["speaker", "play_*"]
    }
  }
}
```
{% include copy-curl.html %}

如果您不提供`fields` 范围，`multi_match` 查询搜索在`index.query. Default_field` 设置，默认为`*`。默认行为是提取有资格的映射中的所有字段[学期-级查询]({{site.url}}{{site.baseurl}}/query-dsl/term/index/)，过滤元数据字段，然后组合所有提取的字段以构建查询。

查询中的最大从句数量在`indices.query.bool.max_clause_count` 设置，默认为1,024。
{:.note}

## 多-匹配查询类型

OpenSearch支持以下多数-匹配查询类型，在内部执行查询的方式上不同：

- [`best_fields`](#best-fields) （默认值）：返回与任何字段匹配的文档。使用`_score` 最好的-匹配字段。
- [`most_fields`](#most-fields)：返回与任何字段匹配的文档。使用每个匹配字段的组合得分。
- [`cross_fields`](#cross-fields)：对待所有领域，就好像它们是一个领域一样。相同的过程字段`analyzer` 并匹配任何领域的单词。
- [`phrase`](#phrase)：运行a`match_phrase` 在每个字段上查询。使用`_score` 最好的-匹配字段。
- [`phrase_prefix`](#phrase-prefix)：运行a`match_phrase_prefix` 在每个字段上查询。使用`_score` 最好的-匹配字段。
- [`bool_prefix`](#boolean-prefix)：运行a`match_bool_prefix` 在每个字段上查询。使用每个匹配字段的组合得分。

## 最好的领域

如果您要搜索两个指定概念的单词，则需要结果两个单词相邻得分更高的结果。

例如，考虑包含以下科学文章的索引：

```json
PUT /articles/_doc/1
{
  "title": "Aurora borealis",
  "description": "Northern lights, or aurora borealis, explained"
}
```
{% include copy-curl.html %}

```json
PUT /articles/_doc/2
{
  "title": "Sun deprivation in the Northern countries",
  "description": "Using fluorescent lights for therapy"
}
```
{% include copy-curl.html %}

您可以搜索包含的文章`northern lights` 在标题或描述中：

```json
GET articles/_search
{
  "query": {
    "multi_match" : {
      "query": "northern lights",
      "type": "best_fields",
      "fields": [ "title", "description" ],
      "tie_breaker": 0.3
    }
  }
}
```
{% include copy-curl.html %}

前面的查询被执行为以下[`dis_max`]({{site.url}}{{site.baseurl}}/query-dsl/compound/disjunction-max/) 用一个`match` 每个字段的查询：

```json
GET /articles/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "title": "northern lights" }},
        { "match": { "description": "northern lights" }}
      ],
      "tie_breaker": 0.3
    }
  }
}
```

结果包含两个文档，但是文档1的评分较高，因为两个单词都在`description` 场地：

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
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.84407747,
    "hits": [
      {
        "_index": "articles",
        "_id": "1",
        "_score": 0.84407747,
        "_source": {
          "title": "Aurora borealis",
          "description": "Northern lights, or aurora borealis, explained"
        }
      },
      {
        "_index": "articles",
        "_id": "2",
        "_score": 0.6322521,
        "_source": {
          "title": "Sun deprivation in the Northern countries",
          "description": "Using fluorescent lights for therapy"
        }
      }
    ]
  }
}
```

这`best_fields` 查询使用最佳分数-匹配字段。如果指定`tie_breaker`，使用以下算法计算得分：

取得最好的成绩-匹配字段并添加（`tie_breaker` *`_score`）适用于所有其他匹配字段。

## 大多数字段

使用`most_fields` 查询包含以不同方式分析的相同文本的多个字段。例如，原始字段可能包含使用的文本`standard` 分析仪和另一个字段可能包含与`english` 分析仪执行驱动：

```json
PUT /articles
{
  "mappings": {
    "properties": {
      "title": { 
        "type": "text",
        "fields": {
          "english": { 
            "type": "text",
            "analyzer": "english"
          }
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

考虑以下两个文档`articles` 指数：

```json
PUT /articles/_doc/1
{
  "title": "Buttered toasts"
}
```
{% include copy-curl.html %}

```json
PUT /articles/_doc/2
{
  "title": "Buttering a toast"
}
```
{% include copy-curl.html %}

这`standard` 分析仪分析标题`Buttered toast` 进入 [`buttered`，`toasts`]和标题`Buttering a toast` 进入 [`buttering`，`a`，`toast`]。另一方面，`english` 分析仪会产生相同的令牌列表[`butter`，`toast`]对于两个标题，由于茎。

您可以使用`most_fields` 查询以返回尽可能多的文档：

```json
GET /articles/_search
{
  "query": {
    "multi_match": {
      "query": "buttered toast",
      "fields": [ 
        "title",
        "title.english"
      ],
      "type": "most_fields" 
    }
  }
}
```
{% include copy-curl.html %}

前面的查询被执行为以下布尔查询：

```json
GET articles/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title": "buttered toasts" }},
        { "match": { "title.english": "buttered toasts" }}
      ]
    }
  }
}
```

为了计算相关性分数，文件的分数`match` 将子句添加在一起，然后将结果除以`match` 条款。

包括`title.english` 字段检索与STEMMed令牌相匹配的第二个文档：

```json
{
  "took": 9,
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
    "max_score": 1.4418206,
    "hits": [
      {
        "_index": "articles",
        "_id": "1",
        "_score": 1.4418206,
        "_source": {
          "title": "Buttered toasts"
        }
      },
      {
        "_index": "articles",
        "_id": "2",
        "_score": 0.09304003,
        "_source": {
          "title": "Buttering a toast"
        }
      }
    ]
  }
}
```

因为两者`title` 和`title.english` 字段匹配第一个文档，其相关性得分更高。

## 操作员和最低应匹配

这`best_fields` 和`most_fields` 查询在字段基础上生成匹配查询（每个字段）。就这样`minimum_should_match` 和`operator` 参数应用于每个字段，通常不是所需的行为。

例如，考虑一个`customers` 带有以下文件的索引：

```json
PUT customers/_doc/1 
{
  "first_name": "John",
  "last_name": "Doe"
}
```
{% include copy-curl.html %}

```json
PUT customers/_doc/2 
{
  "first_name": "Jane",
  "last_name": "Doe"
}
```
{% include copy-curl.html %}

如果您正在寻找`John Doe` 在里面`customers` 索引，您可以构建以下查询：

```json
GET customers/_validate/query?explain
{
  "query": {
    "multi_match" : {
      "query": "John Doe",
      "type": "best_fields",
      "fields": [ "first_name", "last_name" ],
      "operator": "and" 
    }
  }
}
```
{% include copy-curl.html %}

意图`and` 此查询中的操作员是找到匹配的文档`John` 和`Doe`。但是，查询不会返回任何结果。您可以通过运行Validate API来了解如何执行查询：

```json
GET customers/_validate/query?explain
{
  "query": {
    "multi_match" : {
      "query":      "John Doe",
      "type":       "best_fields",
      "fields":     [ "first_name", "last_name" ],
      "operator":   "and" 
    }
  }
}
```
{% include copy-curl.html %}

从响应中，您可以看到查询正在尝试匹配两者`John` 和`Doe` 要么`first_name` 或者`last_name` 场地：

```json
{
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "valid": true,
  "explanations": [
    {
      "index": "customers",
      "valid": true,
      "explanation": "((+first_name:john +first_name:doe) | (+last_name:john +last_name:doe))"
    }
  ]
}
```

因为两个字段都包含两个单词，所以没有返回结果。

跨字段搜索的更好替代方法是使用[`cross_fields`](#cross-fields) 询问。与领域不同-中心`best_fields` 和`most_fields` 查询，`cross_fields` 查询是术语-中心。

## 跨场

使用`cross_fields` 查询跨多个字段搜索数据。例如，如果索引包含客户数据，则客户的名字和姓氏位于不同的字段中。但是，当您搜索`John Doe`，您想收到文件`John` 在里面`first_name` 字段和`Doe` 在里面`last_name` 场地。

这`most_fields` 由于以下问题，查询在这种情况下不起作用：

- 这[`operator` 和`minimum_should_match`](#operator-and-minimum-should-match) 参数是在字段上而不是在术语的基础上应用的。
- 术语频率`first_name` 和`last_name` 字段可能导致意外的结果。例如，如果某人的名字恰好是`Doe`，将其名称的文档推定为更好的匹配，因为此名称不会出现在任何其他文档中。

这`cross_fields` 查询将查询字符串分析到单个术语中，然后在任何字段中搜索每个术语，就好像它们是一个字段一样。

以下是`cross_fields` 查询`John Doe`：

```json
GET /customers/_search
{
  "query": {
    "multi_match" : {
      "query": "John Doe",
      "type": "cross_fields",
      "fields": [ "first_name", "last_name" ],
      "operator": "and"
    }
  }
}
```
{% include copy-curl.html %}

响应包含唯一的文档`John` 和`Doe` 存在：

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
    "max_score": 0.8754687,
    "hits": [
      {
        "_index": "customers",
        "_id": "1",
        "_score": 0.8754687,
        "_source": {
          "first_name": "John",
          "last_name": "Doe"
        }
      }
    ]
  }
}
```

您可以使用Validate API操作来了解如何执行前面的查询：

```json
GET /customers/_validate/query?explain
{
  "query": {
    "multi_match" : {
      "query": "John Doe",
      "type": "cross_fields",
      "fields": [ "first_name", "last_name" ],
      "operator": "and"
    }
  }
}
```
{% include copy-curl.html %}

从响应中，您可以看到查询至少在一个字段中搜索所有术语：

```json
{
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "valid": true,
  "explanations": [
    {
      "index": "customers",
      "valid": true,
      "explanation": "+blended(terms:[last_name:john, first_name:john]) +blended(terms:[last_name:doe, first_name:doe])"
    }
  ]
}
```

因此，混合所有字段的术语频率通过校正差异来解决不同项频率的问题。

这`cross_fields` 查询通常仅在带有一个短字符串字段上有用`boost` 在1.的情况下，在其他情况下，分数不会产生有意义的术语统计融合，这是因为增强，项频率和长度归一化的方式会导致分数。
{:.note}

这`fuzziness` 不支持参数`cross_fields` 查询。
{:.note}

### 分析

这`cross_fields` 查询只能用作术语-具有相同分析仪的字段中的以中心查询。具有相同分析仪的字段被分组在一起，这些组与布尔查询结合在一起。

例如，考虑一个索引`first_name` 和`last_name` 用默认值分析字段`standard`
 分析仪及其`.edge` 用边缘n分析子场-克分析仪：

<details closed markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}

```json
PUT customers
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "my_tokenizer"
        }
      },
      "tokenizer": {
        "my_tokenizer": {
          "type": "edge_ngram",
          "min_gram": 2,
          "max_gram": 10
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "first_name": { 
        "type": "text",
        "fields": {
          "edge": { 
            "type": "text",
            "analyzer": "my_analyzer"
          }
        }
      },
      "last_name": { 
        "type": "text",
        "fields": {
          "edge": { 
            "type": "text",
            "analyzer": "my_analyzer"
          }
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

</delect>

您在`customers` 指数：

```json
PUT /customers/_doc/1
{
  "first": "John",
  "last": "Doe"
}
```
{% include copy-curl.html %}

您可以使用`cross_fields` 查询以跨字段搜索`John Doe`：

```json
GET /customers/_search
{
  "query": {
    "multi_match" : {
      "query": "John",
      "type": "cross_fields",
      "fields": [
        "first_name", "first_name.edge",
        "last_name",  "last_name.edge"
      ]
    }
  }
}
```
{% include copy-curl.html %}

要查看查询的执行方式，您可以运行Validate API：

```json
GET /customers/_validate/query?explain
{
  "query": {
    "multi_match" : {
      "query": "John",
      "type": "cross_fields",
      "fields": [
        "first_name", "first_name.edge",
        "last_name",  "last_name.edge"
      ]
    }
  }
}
```
{% include copy-curl.html %}

回应表明`last_name` 和`first_name` 将字段分组在一起，并将其视为一个字段。同样，`last_name.edge` 和`first_name.edge` 字段被分组在一起，并将其视为一个字段：

```json
{
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "valid": true,
  "explanations": [
    {
      "index": "customers",
      "valid": true,
      "explanation": "(blended(terms:[last_name:john, first_name:john]) | (blended(terms:[last_name.edge:Jo, first_name.edge:Jo]) blended(terms:[last_name.edge:Joh, first_name.edge:Joh]) blended(terms:[last_name.edge:John, first_name.edge:John])))"
    }
  ]
}
```

使用`operator` 或者`minimum_should_match` 具有多个字段组（例如上一个字段）的参数可能会导致在[上一节](#operator-and-minimum-should-match)。为了避免，您可以将上一个查询重写为两个`cross_fields` 子征服与布尔查询相结合并应用`minimum_should_match` 向其中一个子征服：

```json
GET /customers/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "multi_match": {
            "query": "John Doe",
            "type": "cross_fields",
            "fields": [
              "first_name",
              "last_name"
            ],
            "minimum_should_match": "1"
          }
        },
        {
          "multi_match": {
            "query": "John Doe",
            "type": "cross_fields",
            "fields": [
              "first_name.edge",
              "last_name.edge"
            ]
          }
        }
      ]
    }
  }
}
```
{% include copy-curl.html %}

要为所有字段创建一个组，请在查询中指定一个分析仪：

```json
GET customers/_search
{
  "query": {
   "multi_match" : {
      "query": "John Doe",
      "type": "cross_fields",
      "analyzer": "standard", 
      "fields": [ "first_name", "last_name", "*.edge" ]
    }
  }
}
```
{% include copy-curl.html %}

在上一个查询上运行validate API显示了查询的执行方式：

```json
{
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "valid": true,
  "explanations": [
    {
      "index": "customers",
      "valid": true,
      "explanation": "blended(terms:[last_name.edge:john, last_name:john, first_name:john, first_name.edge:john]) blended(terms:[last_name.edge:doe, last_name:doe, first_name:doe, first_name.edge:doe])"
    }
  ]
}
```

## 短语

这`phrase` 查询的行为与[`best_fields`](#best-fields) 查询但使用`match_phrase` 查询而不是`match` 询问。

以下是一个例子`phrase` 查询在[`best_fields`](#best-fields) 部分：

```json
GET articles/_search
{
  "query": {
    "multi_match" : {
      "query": "northern lights",
      "type": "phrase",
      "fields": [ "title", "description" ]
    }
  }
}
```
{% include copy-curl.html %}

前面的查询被执行为以下[`dis_max`]({{site.url}}{{site.baseurl}}/query-dsl/compound/disjunction-max/) 用一个`match_phrase` 每个字段的查询：

```json
GET articles/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match_phrase": { "title": "northern lights" }},
        { "match_phrase": { "description": "northern lights" }}
      ]
    }
  }
}
```

因为默认一个`phrase` 查询仅在以相同顺序出现的术语出现时匹配文本，结果仅返回文档1：

<details closed markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}

```json
{
  "took": 3,
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
    "max_score": 0.84407747,
    "hits": [
      {
        "_index": "articles",
        "_id": "1",
        "_score": 0.84407747,
        "_source": {
          "title": "Aurora borealis",
          "description": "Northern lights, or aurora borealis, explained"
        }
      }
    ]
  }
}
```
</details>

您可以使用`slop` 参数允许查询短语中的单词之间的其他单词。例如，以下查询接受文本作为匹配，如果最多两个单词之间`flourescent` 和`therapy`：

```json
GET articles/_search
{
  "query": {
    "multi_match" : {
      "query": "fluorescent therapy",
      "type": "phrase",
      "fields": [ "title", "description" ],
      "slop": 2
    }
  }
}
```
{% include copy-curl.html %}

响应包含文件2：

<details closed markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}

```json
{
  "took": 3,
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
    "max_score": 0.7003825,
    "hits": [
      {
        "_index": "articles",
        "_id": "2",
        "_score": 0.7003825,
        "_source": {
          "title": "Sun deprivation in the Northern countries",
          "description": "Using fluorescent lights for therapy"
        }
      }
    ]
  }
}
```
</details>

为了`slop` 值小于2，没有返回文档。

这`fuzziness` 不支持参数`phrase` 查询。
{:.note}

## 短语前缀

这`phrase_prefix` 查询的行为与[`phrase`](#phrase) 查询但使用`match_phrase_prefix` 查询而不是`match_phrase` 询问。

以下是一个例子`phrase_prefix` 查询在[`best_fields`](#best-fields) 部分：

```json
GET articles/_search
{
  "query": {
    "multi_match" : {
      "query": "northern light",
      "type": "phrase_prefix",
      "fields": [ "title", "description" ]
    }
  }
}
```
{% include copy-curl.html %}

前面的查询被执行为以下[`dis_max`]({{site.url}}{{site.baseurl}}/query-dsl/compound/disjunction-max/) 用一个`match_phrase_prefix` 每个字段的查询：

```json
GET articles/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match_phrase_prefix": { "title": "northern light" }},
        { "match_phrase_prefix": { "description": "northern light" }}
      ]
    }
  }
}
```

您可以使用`slop` 参数允许查询短语中的单词之间的其他单词。

这`fuzziness` 不支持参数`phrase_prefix` 查询。
{:.note}

## 布尔前缀

这`bool_prefix` 查询分数文档与[`most_fields`](#most-fields) 查询但使用[`match_bool_prefix`]({{site.url}}{{site.baseurl}}/query-dsl/full-text/match-bool-prefix/) 查询而不是`match` 询问。

以下是一个例子`bool_prefix` 查询在[`best_fields`](#best-fields) 部分：

```json
GET articles/_search
{
  "query": {
    "multi_match" : {
      "query": "li northern",
      "type": "bool_prefix",
      "fields": [ "title", "description" ]
    }
  }
}
```
{% include copy-curl.html %}

前面的查询被执行为以下[`dis_max`]({{site.url}}{{site.baseurl}}/query-dsl/compound/disjunction-max/) 用一个`match_bool_prefix` 每个字段的查询：

```json
GET articles/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match_bool_prefix": { "title": "li northern" }},
        { "match_bool_prefix": { "description": "li northern" }}
      ]
    }
  }
}
```

这`fuzziness`，`prefix_length`，`max_expansions`，`fuzzy_rewrite`， 和`fuzzy_transpositions` 支持用于构建项的术语查询的术语的参数，但它们对从最后一项构建的前缀查询没有影响。
{:.note}

## 参数

查询接受以下参数。除所有参数外`query` 是可选的。

范围| 数据类型| 描述
:--- | :--- | :---
`query` | 细绳| 用于搜索的查询字符串。必需的。
`auto_generate_synonyms_phrase_query` | 布尔| 指定是否创建[匹配短语查询]({{site.url}}{{site.baseurl}}/query-dsl/full-text/match-phrase/) 自动用于多人-术语同义词。例如，如果指定`ba,batting average` 作为同义词和搜索`ba`，OpenSearch搜索`ba OR "batting average"` （如果此选项是`true`） 或者`ba OR (batting AND average)` （如果此选项是`false`）。默认为`true`。
`analyzer` | 细绳| 这[分析仪]({{site.url}}{{site.baseurl}}/analyzers/index/) 用于引导查询字符串文本。默认是索引-指定的时间分析仪`default_field`。如果未针对`default_field`， 这`analyzer` 是索引的默认分析仪。
`boost` | 漂浮的-观点| 通过给定的乘数增强子句。对于在复合查询中称量从句有用。[0，1）中的值降低了相关性，并且值大于1的相关性。默认为`1`。
`fields` | 弦数| 搜索字段列表。如果您不提供`fields` 范围，`multi_match` 查询搜索在`index.query. Default_field` 设置，默认为`*`。
`fuzziness` | 细绳| 在确定一个术语是否匹配值时，将一个单词更改为另一个单词所需的字符编辑数量（插入，删除，替换）。例如，`wined` 和`wind` 是1.有效值是非-负整数或`AUTO`。默认，`AUTO`，根据每个学期的长度选择一个值，对于大多数用例，是一个不错的选择。不支持`phrase`，`phrase_prefix`， 和`cross_fields` 查询。
`fuzzy_rewrite` | 细绳| 确定OpenSearch如何重写查询。有效值是`constant_score`，`scoring_boolean`，`constant_score_boolean`，`top_terms_N`，`top_terms_boost_N`， 和`top_terms_blended_freqs_N`。如果是`fuzziness` 参数不是`0`，查询使用`fuzzy_rewrite` 的方法`top_terms_blended_freqs_${max_expansions}` 默认情况下。默认为`constant_score`。
`fuzzy_transpositions` | 布尔| 环境`fuzzy_transpositions` 到`true` （默认）在插入，删除和替代操作中添加相邻字符的互换`fuzziness` 选项。例如，`wind` 和`wnid` 是1`fuzzy_transpositions` 是真的（交换"n" 和"i"）和2如果是错误的（删除"n"， 插入"n"）。如果`fuzzy_transpositions` 是错误的，`rewind` 和`wnid` 距离有相同的距离（2）`wind`，尽管人类越来越多-以中心的看法`wnid` 是一个明显的错别字。对于大多数用例，默认值是一个不错的选择。
`lenient` | 布尔| 环境`lenient` 到`true` 忽略查询和文档字段之间的数据类型不匹配。例如，一个查询字符串`"8.2"` 可以匹配类型字段`float`。默认为`false`。
`max_expansions` | 正整数|  查询可以扩展的最大术语数量。模糊的查询“扩展为”在指定距离内的许多匹配术语`fuzziness`。然后OpenSearch尝试匹配这些术语。默认为`50`。
`minimum_should_match` | 正整数，正数，正百分比，组合| 如果查询字符串包含多个搜索术语，并且您使用`or` 操作员，需要将文档匹配的术语数量被视为匹配。例如，如果`minimum_should_match` 是2，`wind often rising` 不匹配`The Wind Rises.` 如果`minimum_should_match` 是`1`， 它匹配。有关详细信息，请参阅[最低应匹配]({{site.url}}{{site.baseurl}}/query-dsl/minimum-should-match/)。
`operator` | 细绳| 如果查询字符串包含多个搜索术语，则所有术语是否需要匹配（`AND`）或只需要一个术语匹配（`OR`）文档被视为匹配项。有效值是：<br>- `OR`：字符串`to be` 被解释为`to OR be`<br>- `AND`：字符串`to be` 被解释为`to AND be`<br>默认值为`OR`。
`prefix_length` | 非-负整数| 在模糊性中未考虑的领先角色的数量。默认为`0`。
`slop` | `0` （默认）或正整数| 控制查询中的单词可能会被误解的程度，并且仍然被认为是匹配的程度。来自[Lucene文档](https://lucene.apache.org/core/8_9_0/core/org/apache/lucene/search/PhraseQuery.html#getSlop--)："The number of other words permitted between words in query phrase. For example, to switch the order of two words requires two moves (the first move places the words atop one another), so to permit reorderings of phrases, the slop must be at least two. A value of zero requires an exact match." 支持`phrase` 和`phrase_prefix` 查询类型。
`tie_breaker` | 漂浮的-观点| 0到1.0之间的因素，用于使匹配多个查询子句的文档给予更大的权重。有关更多信息，请参阅[这`tie_breaker` 范围`](#the-tie_breaker-parameter)。
`类型` | String | The multi-match query type. Valid values are `best_fields`, `mosp_fields`, `cross_fields`, `短语`, `phrase_prefix`, `bool_prefix`. Default is `best_fields`.
`ZERO_TERMS_QUERY` | String | In some cases, the analyzer removes all terms from a query string. For example, the `停止` analyzer removes all terms from the string `但是这个`. In those cases, `ZERO_TERMS_QUERY` specifies whether to match no documents (`没有任何`) or all documents (`全部`). Valid values are `没有任何` and `全部`. Default is `没有任何`。

这`fuzziness` 不支持参数`phrase`，`phrase_prefix`， 和`cross_fields` 查询。
{:.note}

这`slop` 参数仅支持`phrase` 和`phrase_prefix` 查询。
{:.note}

### 这`tie_breaker` 范围

每个学期-级别混合查询将文档分数计算为小组中任何字段返回的最佳分数。所有混合查询的分数都会添加在一起以产生最终分数。您可以通过使用`tie_breaker` 范围。这`tie_breaker` 参数接受以下值：

- 0.0（默认`best_fields`，`cross_fields`，`phrase`， 和`phrase_prefix` 查询）：以组中任何字段返回的单一最佳分数。
- 1.0（默认`most_fields` 和`bool_prefix` 查询）：添加组中所有字段的分数。
- 浮动-（0，1）范围中的点值：取得最佳的最佳分数-匹配字段并添加（`tie_breaker` *`_score`）对于所有其他匹配字段


---
layout: default
title: 匹配布尔前缀
parent: 全文查询
grand_parent: 查询DSL
nav_order: 20
---

# 匹配布尔前缀查询

这`match_bool_prefix` 查询分析提供的搜索字符串并创建一个[布尔查询]({{site.url}}{{site.baseurl}}/query-dsl/compound/bool/) 从字符串的术语中。它使用每个术语，除了最后一个术语作为匹配的整个单词。最后一项用作前缀。这`match_bool_prefix` 查询返回包含整个的文档-以任何顺序从前缀术语开始的单词术语或术语。

以下示例显示了一个基本`match_bool_prefix` 询问：

```json
GET _search
{
  "query": {
    "match_bool_prefix": {
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
    "match_bool_prefix": {
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

下列`match_bool_prefix` 查询搜索整个单词`rises` 和以`wi`，按任何顺序：

```json
GET testindex/_search
{
  "query": {
    "match_bool_prefix": {
      "title": "rises wi"
    }
  }
}
```
{% include copy-curl.html %}

前面的查询等同于以下布尔查询：

```json
GET testindex/_search
{
  "query": {
    "bool" : {
      "should": [
        { "term": { "title": "rises" }},
        { "prefix": { "title": "wi"}}
      ]
    }
  }
}
```

响应包含两个文档：

<details closed markdown="block">
  <summary>
    回复
  </summary>
  {： 。文本-三角洲}

```json
{
  "took": 15,
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
    "max_score": 1.73617,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 1.73617,
        "_source": {
          "title": "The wind rises"
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 1,
        "_source": {
          "title": "Gone with the wind"
        }
      }
    ]
  }
}
```

</delect>

## 这`match_bool_prefix` 和`match_phrase_prefix` 查询

这`match_bool_prefix` 查询与任何位置的术语匹配，而`match_phrase_prefix` 查询将术语与整个短语匹配。为了说明差异，再次考虑`match_bool_prefix` 从上一节查询：

```json
GET testindex/_search
{
  "query": {
    "match_bool_prefix": {
      "title": "rises wi"
    }
  }
}
```
{% include copy-curl.html %}

两个都`The wind rises` 和`Gone with the wind` 匹配搜索词，因此查询返回两个文档。

现在运行一个`match_phrase_prefix` 在同一索引上查询：

```json
GET testindex/_search
{
  "query": {
    "match_phrase_prefix": {
      "title": "rises wi"
    }
  }
}
```
{% include copy-curl.html %}

响应没有返回文档，因为没有任何文档包含短语`rises wi` 按照指定的顺序。

## 分析仪

默认情况下，当您在`text` 字段，使用与字段关联的索引分析仪分析搜索文本。您可以在`analyzer` 范围：

```json
GET testindex/_search
{
  "query": {
    "match_bool_prefix": {
      "title": {
        "query": "rise the wi",
        "analyzer": "stop"
      }
    }
  }
}
```
{% include copy-curl.html %}
 
## 参数

查询接受字段的名称（`<field>`）作为顶部-级别参数：

```json
GET _search
{
  "query": {
    "match_bool_prefix": {
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
`query` | 细绳| 文本，数字，布尔值或用于搜索的日期。必需的。
`analyzer` | 细绳| 这[分析仪]({{site.url}}{{site.baseurl}}/analyzers/index/) 用于引导查询字符串文本。默认是索引-指定的时间分析仪`default_field`。如果未针对`default_field`， 这`analyzer` 是索引的默认分析仪。
`fuzziness` | `AUTO`，`0`，或一个积极的整数| 在确定一个术语是否匹配值时，将一个单词更改为另一个单词所需的字符编辑数量（插入，删除，替换）。例如，`wined` 和`wind` 是1.默认`AUTO`，根据每个学期的长度选择一个值，对于大多数用例，是一个不错的选择。
`fuzzy_rewrite` | 细绳| 确定OpenSearch如何重写查询。有效值是`constant_score`，`scoring_boolean`，`constant_score_boolean`，`top_terms_N`，`top_terms_boost_N`， 和`top_terms_blended_freqs_N`。如果是`fuzziness` 参数不是`0`，查询使用`fuzzy_rewrite` 的方法`top_terms_blended_freqs_${max_expansions}` 默认情况下。默认为`constant_score`。
`fuzzy_transpositions` | 布尔| 环境`fuzzy_transpositions` 到`true` （默认）在插入，删除和替代操作中添加相邻字符的互换`fuzziness` 选项。例如，`wind` 和`wnid` 是1`fuzzy_transpositions` 是真的（交换"n" 和"i"）和2如果是错误的（删除"n"， 插入"n"）。如果`fuzzy_transpositions` 是错误的，`rewind` 和`wnid` 距离有相同的距离（2）`wind`，尽管人类越来越多-以中心的看法`wnid` 是一个明显的错别字。对于大多数用例，默认值是一个不错的选择。
`max_expansions` | 正整数|  查询可以扩展的最大术语数量。模糊的查询“扩展为”在指定距离内的许多匹配术语`fuzziness`。然后OpenSearch尝试匹配这些术语。默认为`50`。
`minimum_should_match` | 正整数，正数，正百分比，组合| 如果查询字符串包含多个搜索术语，并且您使用`or` 操作员，需要将文档匹配的术语数量被视为匹配。例如，如果`minimum_should_match` 是2，`wind often rising` 不匹配`The Wind Rises.` 如果`minimum_should_match` 是`1`， 它匹配。有关详细信息，请参阅[最低应匹配]({{site.url}}{{site.baseurl}}/query-dsl/minimum-should-match/)。
`operator` | 细绳| 如果查询字符串包含多个搜索术语，则所有术语是否需要匹配（`and`）或只需要一个术语匹配（`or`）文档被视为匹配项。有效值是`or` 和`and`。默认为`or`。
`prefix_length` | 非-负整数| 在模糊性中未考虑的领先角色的数量。默认为`0`。

这`fuzziness`，`fuzzy_transpositions`，`fuzzy_rewrite`，`max_expansions`， 和`prefix_length` 参数可以应用于除最后一项以外的所有术语构建的术语次数。它们对最后一届构建的前缀查询没有任何影响。
{： 。笔记


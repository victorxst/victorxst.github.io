---
layout: default
title: 简单查询字符串
parent: 全文查询
grand_parent: 查询DSL
nav_order: 70
---

# 简单查询字符串查询

使用`simple_query_string` 键入直接在查询字符串中通过正则表达式描述的多个参数。简单查询字符串的语法不如查询字符串，因为它会丢弃字符串的任何无效部分，并且不会返回无效语法的错误。

此查询使用[简单的语法](#simple-query-string-syntax) 根据特殊操作员解析查询字符串，然后将字符串分为术语。解析后，查询分析每个术语，然后返回匹配的文档。

以下查询在`title` 场地：

```json
GET _search
{
  "query": {
    "simple_query_string": {
      "query": "\"rises wind the\"~4 | *ising~2",
      "fields": ["title"]
    }
  }
}
```
{% include copy-curl.html %}

## 简单查询字符串语法

查询字符串由_terms_和_operators _组成。一个术语是一个单词（例如，在查询中`wind rises`，这些条款是`wind` 和`rises`）。如果几个术语被引号包围，则将其视为一个短语，其中单词按照它们出现的顺序进行（例如，`"wind rises"`）。运营商，例如`+`，`|`， 和`-` 指定用于解释查询字符串中文本的布尔逻辑。

## 操作员

简单查询字符串语法支持以下运算符。

操作员| 描述
:--- | :---
`+` | 充当`AND` 操作员。
`|` | 充当`OR` 操作员。
`*` | 在学期结束时使用时，表示前缀查询。
`"` | 将几个术语包裹在短语中（例如，`"wind rises"`）。
`(`，`)` | 将条款以优先级为单位（例如，`wind + (rises | rising)`）。
`~n` | 术语后使用时（例如，`wnid~3`）， 套`fuzziness`。用短语使用时，设置`slop`。
`-` | 否定该术语。

所有前一个操作员都是保留字符。要将它们称为原始角色而不是操作员，请以后斜击逃脱其中任何一个。发送JSON请求时，请使用`\\` 为了逃避保留的字符（因为后斜切的字符本身是保留的，因此您必须与另一个后斜线逃脱后斜线）。

## 默认操作员

默认操作员是`OR` （除非您设置`default_operator` 到`AND`）。默认操作员决定了总体查询行为。例如，考虑包含以下文档的索引：

```json
PUT /customers/_doc/1
{
  "first_name":"Amber",
  "last_name":"Duke",
  "address":"880 Holmes Lane"
}
```
{% include copy-curl.html %}

```json
PUT /customers/_doc/2
{
  "first_name":"Hattie",
  "last_name":"Bond",
  "address":"671 Bristol Street"
}
```
{% include copy-curl.html %}

```json
PUT /customers/_doc/3
{
  "first_name":"Nanette",
  "last_name":"Bates",
  "address":"789 Madison St"
}
```
{% include copy-curl.html %}

```json
PUT /customers/_doc/4
{
  "first_name":"Dale",
  "last_name":"Amber",
  "address":"467 Hutchinson Court"
}
```
{% include copy-curl.html %}

以下查询尝试查找文档，地址包含单词`street` 或者`st` 并且不包含这个词`madison`：

```json
GET /customers/_search
{
  "query": {
    "simple_query_string": {
      "fields": [ "address" ],
      "query": "street st -madison"
    }
  }
}

```
{% include copy-curl.html %}

但是，结果不仅包括预期文件，而且包括所有四个文件：

<详细信息关闭的markdown ="block">
  <summary>
    回复
  </summary>
  {： 。文本-三角洲}

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
      "value": 4,
      "relation": "eq"
    },
    "max_score": 2.2039728,
    "hits": [
      {
        "_index": "customers",
        "_id": "2",
        "_score": 2.2039728,
        "_source": {
          "first_name": "Hattie",
          "last_name": "Bond",
          "address": "671 Bristol Street"
        }
      },
      {
        "_index": "customers",
        "_id": "3",
        "_score": 1.2039728,
        "_source": {
          "first_name": "Nanette",
          "last_name": "Bates",
          "address": "789 Madison St"
        }
      },
      {
        "_index": "customers",
        "_id": "1",
        "_score": 1,
        "_source": {
          "first_name": "Amber",
          "last_name": "Duke",
          "address": "880 Holmes Lane"
        }
      },
      {
        "_index": "customers",
        "_id": "4",
        "_score": 1,
        "_source": {
          "first_name": "Dale",
          "last_name": "Amber",
          "address": "467 Hutchinson Court"
        }
      }
    ]
  }
}
```
</details>

因为默认操作员是`OR`，此查询包括包含单词的文档`street` 或者`st` （文档2和3）以及不包含单词的文档`madison` （文档1和4）。

正确表达查询意图，之前`-madison` 和`+`：

```json
GET /customers/_search
{
  "query": {
    "simple_query_string": {
      "fields": [ "address" ],
      "query": "street st +-madison"
    }
  }
}
```
{% include copy-curl.html %}

或者，指定`AND` 作为默认运算符，并使用分离作为单词`street` 和`st`：

```json
GET /customers/_search
{
  "query": {
    "simple_query_string": {
      "fields": [ "address" ],
      "query": "st|street -madison",
      "default_operator": "AND"
    }
  }
}
```
{% include copy-curl.html %}

上述查询返回文档2：

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
    "max_score": 2.2039728,
    "hits": [
      {
        "_index": "customers",
        "_id": "2",
        "_score": 2.2039728,
        "_source": {
          "first_name": "Hattie",
          "last_name": "Bond",
          "address": "671 Bristol Street"
        }
      }
    ]
  }
}
```
</details>

## 限制操作员

为了限制简单查询字符串解析器的受支持的操作员，包括您要支持的操作员`|`， 在里面`flags` 范围。例如，以下查询仅启用`OR`，`AND`， 和`FUZZY` 操作员：

```json
GET /customers/_search
{
  "query": {
    "simple_query_string": {
      "fields": [ "address" ],
      "query": "bristol | madison +stre~2",
      "flags": "OR|AND|FUZZY"
    }
  }
}
```
{% include copy-curl.html %}

下表列出了所有可用的操作员标志。

旗帜| 描述
:--- | :--- 
`ALL` （默认）| 启用所有操作员。
`AND` | 启用`+` （（`AND`） 操作员。
`ESCAPE` | 启用`\` 作为逃生角色。
`FUZZY` | 启用`~n` 一个词后的操作员，哪里`n` 是表示匹配允许的编辑距离的整数。
`NEAR` | 启用`~n` 一句话后的操作员，那里`n` 是匹配令牌之间允许的最大位置数。与...一样`SLOP`。
`NONE` | 禁用所有操作员。
`NOT` | 启用`-` （（`NOT`） 操作员。
`OR` | 启用`|` （（`OR`） 操作员。
`PHRASE` | 启用`"` （引号）短语搜索。
`PRECEDENCE` | 启用`(` 和`)` （括号）操作员优先的操作员。
`PREFIX` | 启用`*` （前缀）操作员。
`SLOP` | 启用`~n` 一句话后的操作员，那里`n` 是匹配令牌之间允许的最大位置数。与...一样`NEAR`。
`WHITESPACE` | 启用白色空间字符作为拆分文本的字符。

## 通配符表达

您可以使用`*` 特殊字符，替代零或更多字符。例如，以下所有字段中的以下查询搜索以结尾`name`：

```json
GET /customers/_search
{
  "query": {
    "simple_query_string" : {
      "query":    "Amber Bond",
      "fields": [ "*name" ] 
    }
  }
}
```
{% include copy-curl.html %}

## 提升

使用套件（`^`）增强操作员通过乘数提高字段的相关性分数。[0，1）中的值降低了相关性，并且值大于1的相关性。默认为`1`。

例如，以下查询搜索`first_name` 和`last_name` 田野和提升来自`first_name` 田地为2倍：

```json
GET /customers/_search
{
  "query": {
    "simple_query_string" : {
      "query":    "Amber",
      "fields": [ "first_name^2", "last_name" ] 
    }
  }
}
```
{% include copy-curl.html %}

## 多-位置令牌

用于多-位置令牌，简单查询字符串创建一个[匹配短语查询]({{site.url}}{{site.baseurl}}/query-dsl/full-text/match-phrase/)。因此，如果您指定`ml, machine learning` 作为同义词和搜索`ml`，OpenSearch搜索`ml OR "machine learning"`。

另外，您可以匹配多匹马-使用连词定位令牌。如果您设置`auto_generate_synonyms_phrase_query` 到`false`，OpenSearch搜索`ml OR (machine AND learning)`。

例如，以下查询搜索文本`ml models` 并指定不自动-为每个同义词生成匹配短语查询：

```json
GET /testindex/_search
{
  "query": {
    "simple_query_string": {
      "fields": ["title"],
      "query": "ml models",
      "auto_generate_synonyms_phrase_query": false
    }
  }
}
```
{% include copy-curl.html %}

对于此查询，OpenSearch创建以下布尔查询：`(ml OR (machine AND learning)) models`。

## 参数

下表列出了顶部-级别参数`simple_query_string` 查询支持。除所有参数外`query` 是可选的。

范围| 数据类型| 描述
:--- | :--- | :---
`query`| 细绳| 可能包含表达式的文本[简单查询字符串语法](#simple-query-string-syntax) 用于搜索。必需的。
`analyze_wildcard` | 布尔| 指定OpenSearch是否应该尝试分析通配符术语。默认为`false`。
`analyzer` | 细绳| 该分析仪用于标记查询字符串文本。默认是索引-指定的时间分析仪`default_field`。如果未针对`default_field`， 这`analyzer` 是索引的默认分析仪。
`auto_generate_synonyms_phrase_query` | 布尔| 指定是否创建[match_phrase查询]({{site.url}}{{site.baseurl}}/query-dsl/full-text/match/) 自动用于多人-术语同义词。默认为`true`。
`default_operator`| 细绳| 如果查询字符串包含多个搜索术语，则所有术语是否需要匹配（`AND`）或只需要一个术语匹配（`OR`）文档被视为匹配项。有效值是：<br>- `OR`：字符串`to be` 被解释为`to OR be`<br>- `AND`：字符串`to be` 被解释为`to AND be`<br>默认值为`OR`。
`fields` | 字符串数组| 要搜索的字段列表（例如，`"fields": ["title^4", "description"]`）。支持通配符。如果未指定，则默认为`index.query. Default_field` 设置，默认为`["*"]`。可以同时搜索的最大字段数量由`indices.query.bool.max_clause_count`，默认情况下为1,024。
`flags` | 细绳| A`|`-划界字符串[标志]({{site.baseurl}}/query-dsl/full-text/simple-query-string/) 启用（例如，`AND|OR|NOT`）。默认为`ALL`。您可以明确设置`default_field`。例如，要返回所有标题，请将其设置为`"default_field": "title"`。
`fuzzy_max_expansions` | 正整数| 查询可以扩展的最大术语数量。模糊的查询“扩展为”在指定距离内的许多匹配术语`fuzziness`。然后OpenSearch尝试匹配这些术语。默认为`50`。
`fuzzy_transpositions` | 布尔| 环境`fuzzy_transpositions` 到`true` （默认）在插入，删除和替代操作中添加相邻字符的互换`fuzziness` 选项。例如，`wind` 和`wnid` 是1`fuzzy_transpositions` 是真的（交换"n" 和"i"）和2如果是错误的（删除"n"， 插入"n"）。如果`fuzzy_transpositions` 是错误的，`rewind` 和`wnid` 距离有相同的距离（2）`wind`，尽管人类越来越多-以中心的看法`wnid` 是一个明显的错别字。对于大多数用例，默认值是一个不错的选择。
`fuzzy_prefix_length`| 整数| 开始匹配的开始角色数量保持不变。默认值为0。
`lenient` | 布尔| 环境`lenient` 到`true` 忽略查询和文档字段之间的数据类型不匹配。例如，一个查询字符串`"8.2"` 可以匹配类型字段`float`。默认为`false`。
`minimum_should_match` | 正整数，正数，正百分比，组合| 如果查询字符串包含多个搜索术语，并且您使用`or` 操作员，需要将文档匹配的术语数量被视为匹配。例如，如果`minimum_should_match` 是2，`wind often rising` 不匹配`The Wind Rises.` 如果`minimum_should_match` 是`1`， 它匹配。有关详细信息，请参阅[最低应匹配]({{site.url}}{{site.baseurl}}/query-dsl/minimum-should-match/)。
`quote_field_suffix` | 细绳| 此选项支持使用与非非非分析方法搜索精确匹配（用引号包围）-确切的匹配使用。例如，如果`quote_field_suffix` 是`.exact` 然后您搜索`\"lightly\"` 在里面`title` 字段，openSearch搜索单词`lightly` 在里面`title.exact` 场地。第二个字段可能使用其他类型（例如，`keyword` 而不是`text`）或其他分析仪


---
layout: default
title: 请求参数
parent: 全文查询
grand_parent: 查询DSL
nav_order: 60

redirect_from:
  - /opensearch/query-dsl/full-text/query-string/
  - /query-dsl/query-dsl/full-text/query-string/
---

# 查询字符串查询

A`query_string` 查询基于[查询字符串语法](#query-string-syntax)。它提供了创建功能强大而简洁的查询，这些查询可以包含通配符并搜索多个字段。

搜索`query_string` 查询不返回嵌套文档。要搜索嵌套字段，请使用[`nested` 询问]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/nested/)。
{:.note}

查询字符串查询具有严格的语法，并在无效的语法中返回错误。因此，对于搜索框应用程序，它不能很好地工作。对于不太严格的替代方案，请考虑使用[`simple_query_string` 询问]({{site.url}}{{site.baseurl}}/query-dsl/full-text/simple-query-string/)。如果您不需要查询语法支持，请使用[`match` 询问]({{site.url}}{{site.baseurl}}/query-dsl/full-text/match/)。
{: .important}

## 查询字符串语法

查询字符串语法基于[apache lucene查询语法](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html)。

您可以在以下情况下使用查询字符串语法：

1. 在一个`query_string` 查询，例如：
    ```json
    GET _search
    {
      "query": {
        "query_string": {
          "query": "the wind AND (rises OR rising)"
        }
      }
    }
    ```
    {% include copy-curl.html %}

1. 在OpenSearch仪表板的Discover应用程序中，如果关闭DQL，如下图所示。
  ![在OpenSearch仪表板中使用查询字符串语法Discover]({{site.url}}{{site.baseurl}}/images/discover-lucene-syntax.png)
  有关更多信息，请参阅[发现]({{site.url}}{{site.baseurl}}/dashboards/discover/index-discover/)。

1. 如果您使用HTTP请求查询参数进行搜索，例如：
  ```json
    GET _search?q=wind
  ```

A query string consists of _terms_ and _operators_. A term is a single word (for example, in the query `风升`, the terms are `风` and `上升`). If several terms are surrounded by quotation marks, they are treated as one phrase where words are marched in the order they appear (for example, `"wind rises"`). Operators (such as `或者`, `和`, and `不是`）指定用于解释查询字符串中文本的布尔逻辑。

本节中的示例使用包含以下映射和文档的索引：

```json
PUT /testindex
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

```json
PUT /testindex/_doc/1
{
  "title": "The wind rises"
}
```
{% include copy-curl.html %}

```json
PUT /testindex/_doc/2
{
  "title": "Gone with the wind",
  "description": "A 1939 American epic historical film"
}
```
{% include copy-curl.html %}

```json
PUT /testindex/_doc/3
{
  "title": "Windy city"
}
```
{% include copy-curl.html %}

```json
PUT /testindex/_doc/4
{
  "article title": "Wind turbines"
}
```
{% include copy-curl.html %}

## 保留字符

以下是查询字符串查询的保留字符的列表：

`+`，`-`，`=`，`&&`，`||`，`>`，`<`，`!`，`(`，`)`，`{`，`}`，`[`，`]`，`^`，`"`，`~`，`*`，`?`，`:`，`\`，`/`

逃避保留角色带有后斜切（`\`）。发送JSON请求时，请使用双重倾斜（`\\`）要逃脱保留的字符（因为后斜切字符本身是保留的，您必须与另一个后斜线逃脱后斜线）。
{:.tip}

例如，搜索表达式`2*3`，指定查询字符串：`2\\*3`：

```json
GET /testindex/_search
{
 "query": {
    "query_string": {
      "query": "title: 2\\*3"
    }
  }
}
```
{% include copy-curl.html %}

这`>` 和`<` 迹象无法逃脱。它们被解释为范围查询。
{: .important}

## 白空间字符和空查询

白空间字符不被视为操作员。如果查询字符串为空或仅包含白空间字符，则查询不会返回结果。

## 字段名称

在结肠前指定字段名称。下表包含带字段名称的示例查询。

查询`query_string` 询问| 查询发现| 文档匹配的标准| 匹配的文档`testindex` 指数
:--- | :--- | :--- | :---
`title: wind` | `title: wind` | 这`title` 字段包含这个词`wind`。| 1，2
`title: (wind OR windy)` | `title: (wind OR windy)` | 这`title` 字段包含这个词`wind` 或单词`windy`。| 1、2、3
`title: \"wind rises\"` | `title: "wind rises"` | 这`title` 字段包含短语`wind rises`。逃脱引号带有后背。| 1
`article\\ title: wind` | `article\ title: wind` | 这`article title` 字段包含这个词`wind`。避开空间角色，后卫。| 4
`title.\\*: rise` | `title.\*: rise` | 每个领域都以`title.` （在此示例中，`title.english`）包含这个词`rise`。逃脱后斜线的通配符角色。| 1
`_exists_: description` | `_exists_: description` | 场`description` 存在。| 2

## 通配符表达

您可以使用特殊字符指定通配符表达式：`?` 取代一个字符，`*` 替换零或更多字符。

#### 例子

以下查询搜索包含单词的标题`gone` 以及一个包含一个单词开头的描述`hist`：

```json
GET /testindex/_search
{
 "query": {
    "query_string": {
      "query": "title: gone AND description: hist*"
    }
  }
}
```
{% include copy-curl.html %}

通配符查询可以使用大量内存，从而降低性能。一个单词开头的通配符（例如，`*cal`）是最昂贵的，因为此类通配符上的匹配文档需要检查索引中的所有术语。禁用领先的通配符，设置`allow_leading_wildcard` 到`false`。
{: .warning}

为了效率，纯通配符，例如`*` 被重写为`exists` 查询。因此，`description: *` 通配符将匹配包含空值的文档`description` 字段，但将不匹配文档`description` 字段要么缺少或有一个`null` 价值。

如果您设置`analyze_wildcard` 到`true`，OpenSearch将分析以一个结尾的查询`*` （例如`hist*`）。因此，OpenSearch将通过在第一个N上进行精确匹配来构建一个布尔查询，其中包括由此产生的令牌-最后一个令牌上有1个令牌和前缀匹配。

## 常用表达

要在查询字符串中指定正则表达模式，请用前斜线包围它们（`/`）， 例如`title: /w[a-z]nd/`。

这`allow_leading_wildcard` 参数不适用于正则表达式。例如，一个查询字符串，例如`/.*d/` 将检查索引中的所有术语。
{: .important}

## 模糊

您可以使用模糊查询`~` 例如，操作员`title: rise~`。

查询搜索在最大允许的编辑距离内包含类似于搜索词的术语的文档。编辑距离定义为[达米拉（Damerau）-Levenshtein距离](https://en.wikipedia.org/wiki/Damerau%E2%80%93Levenshtein_distance)，测量一个-字符更改（插入，删除，替换或换位）才能将一个术语更改为另一个术语。

默认编辑距离为2，应捕获80％的拼写错误。要更改默认的编辑距离，请在`~` 操作员。例如，将编辑距离设置为`1`，使用查询`title: rise~1`。

不要混合模糊和通配符操作员。如果您同时指定模糊和通配符操作员，则不会应用其中一位操作员。例如，如果您可以搜索`wnid*~1`，通配符操作员`*` 将应用，但模糊操作员`~1` 不会应用。
{: .important}

## 接近查询

接近查询不需要搜索短语按指定顺序。它允许短语中的单词按不同的顺序或其他单词分开。接近查询指定短语中单词的最大编辑距离。例如，以下查询允许在指定短语中匹配单词时的编辑距离为4：

```json
GET /testindex/_search
{
 "query": {
    "query_string": {
      "query": "title: \"wind gone\"~4"
    }
  }
}
```
{% include copy-curl.html %}

当OpenSearch匹配文档时，文档中的单词越接近查询中指定的单词顺序（编辑距离越少），文档的相关性分数就越高。

## 范围

要指定数字，字符串或日期字段的范围，请使用方括号（`[min TO max]`）对于包容性范围和卷曲牙套（`{min TO max}`）独家范围。您还可以混合方括号和卷曲支架以包括或排除下部和上限（例如，`{min TO max]`）。

日期范围的日期必须以您在映射包含日期的字段时使用的格式提供。有关支持日期格式的更多信息，请参见[格式]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/date/#formats)。

下表提供了范围的语法示例。

数据类型| 询问| 请求参数
:--- | :--- | :---
数字| 帐号从1到15的文件，包括。| `account_number: [1 TO 15]` 或<br>`account_number: (>=1 AND <=15)` 或<br>`account_number: (+>=1 +<=15)`
| 帐号为15岁以上的文件。| `account_number: [15 TO *]` 或<br>`account_number: >=15` （请注意之后没有空间`>=` 符号）
细绳| 姓氏从包含在内的贝茨到杜克大学的文档。| `lastname: [Bates TO Duke}` 或<br>`lastname: (>=Bates AND <Duke)`
| 姓氏在字母顺序上之前的文档。| `lastname: {* TO Bates}` 或<br>`lastname: <Bates` （请注意之后没有空间`<` 符号）
日期| 发布日期的文档在03/21/2023和09/25/2023（包括）。| `release_date: [03/21/2023 TO 09/25/2023]`

作为在查询字符串中指定范围的替代方案，您可以使用[范围查询]({{site.url}}{{site.baseurl}}/query-dsl/term/range/)，提供更可靠的语法。
{:.tip}

## 提升

使用套件（`^`）增强操作员以乘数提高文档的相关性分数。[0，1）中的值降低了相关性，并且值大于1的相关性。默认为`1`。

下表提供了提升示例。

类型| 描述| 请求参数
:--- | :--- | :---
单词提升| 查找所有包含该词的地址`street` 并提高包含这个词的人`Madison`。| `address: Madison^2 street`
短语提升| 查找包含短语标题的文档`wind rises`，由2增强。| `title: \"wind rises\"^2` 
| 找到包含单词的标题的文档`wind rises`并增加包含短语的文档`wind rises` 由2。| `title: (wind rises)^2`

## 布尔运营商

当您在查询中提供搜索术语时，默认情况下，查询返回包含至少一个提供条款的文档。您可以使用`default_operator` 参数为所有条款指定操作员。因此，如果您设置`default_operator` 到`AND`，所有条款都需要，而如果将其设置为`OR`，所有条款都是可选的。

### `+` 和`-` 操作员

如果您希望对所需和可选术语进行更多的颗粒状控制，则可以使用`+` 和`-` 操作员。这`+` 操作员将术语遵循要求，而`-` 操作员不包括以下术语。

例如，在查询字符串中`title: (gone +wind -turbines)` 指定该术语`gone` 是可选的，该术语`wind` 必须在场和术语`turbines` 不得在匹配文档的标题中出现：

```json
GET /testindex/_search
{
 "query": {
    "query_string": {
      "query": "title: (gone +wind -turbines)"
    }
  }
}
```
{% include copy-curl.html %}

查询返回两个匹配文档：

```json
{
  "_index": "testindex",
  "_id": "2",
  "_score": 1.3159468,
  "_source": {
    "title": "Gone with the wind",
    "description": "A 1939 American epic historical film"
  }
},
{
  "_index": "testindex",
  "_id": "1",
  "_score": 0.3438858,
  "_source": {
    "title": "The wind rises"
  }
}
```

前面的查询等同于以下布尔查询：

```json
GET testindex/_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "title": "wind"
        }
      },
      "should": {
        "match": {
          "title": "gone"
        }
      },
      "must_not": {
        "match": {
          "title": "turbines"
        }
      }
    }
  }
}
```

### 常规布尔运营商

另外，您可以使用以下布尔运营商：`AND`，`&&`，`OR`，`||`，`NOT`，`!`。但是，这些操作员不遵循优先规则，因此您必须使用括号来指定使用多个布尔运算符时指定优先级。例如，查询字符串`title: (gone +wind -turbines)` 可以使用布尔运算符如下重写：

`title: ((gone AND wind) OR wind) AND NOT turbines`

运行以下查询，其中包含重写的查询字符串：

```json
GET testindex/_search
{
 "query": {
    "query_string": {
      "query": "title: ((gone AND wind) OR wind) AND NOT turbines"
    }
  }
}
```
{% include copy-curl.html %}

查询返回与使用的查询相同的结果`+` 和`-` 操作员。但是，请注意，匹配文档的相关性得分与先前结果中的相关性分数不同：

```json
{
  "_index": "testindex",
  "_id": "2",
  "_score": 1.6166971,
  "_source": {
    "title": "Gone with the wind",
    "description": "A 1939 American epic historical film"
  }
},
{
  "_index": "testindex",
  "_id": "1",
  "_score": 0.3438858,
  "_source": {
    "title": "The wind rises"
  }
}
```
{% include copy-curl.html %}

## 分组

使用括号组将多个子句或术语分组为子查询。例如，以下查询搜索包含单词的文档`gone` 或者`rises` 必须包含这个词`wind` 在标题中：

```json
GET testindex/_search
{
 "query": {
    "query_string": {
      "query": "title: (gone OR rises) AND wind"
    }
  }
}
```

结果包含两个匹配文档：

```json
{
  "_index": "testindex",
  "_id": "1",
  "_score": 1.5046883,
  "_source": {
    "title": "The wind rises"
  }
},
{
  "_index": "testindex",
  "_id": "2",
  "_score": 1.3159468,
  "_source": {
    "title": "Gone with the wind",
    "description": "A 1939 American epic historical film"
  }
}
```

您也可以使用分组来提高子查询结果或定位指定字段，例如`title:(gone AND wind) description:(historical film)^2`。

## 搜索多个字段

要搜索多个字段，请使用`fields` 范围。当您提供`fields` 参数，查询被重写为`field_1: query OR field_2: query ...`。

例如，以下查询搜索术语`wind` 或者`film` 在里面`title` 和`description` 字段：

```json
GET testindex/_search
{
  "query": {
    "query_string": {
      "fields": [ "title", "description" ],
      "query": "wind AND film"
    }
  }
}
```
{% include copy-curl.html %}

前面的查询等同于以下查询，该查询不提供`fields` 范围：

```json
GET testindex/_search
{
  "query": {
    "query_string": {
      "query": "(title:wind OR description:wind) AND (title:film OR description:film)"
    }
  }
}
```

### 搜索一个字段的多个子场

要搜索一个字段的所有内部字段，您可以使用通配符。例如，在`address` 字段，使用以下查询：

```json
GET /testindex/_search
{
  "query": {
    "query_string" : {
      "fields" : ["address.*"],
      "query" : "New AND (York OR Jersey)"
    }
  }
}
```
{% include copy-curl.html %}

前面的查询等同于以下查询，该查询不提供`fields` 参数（请注意`*` 被逃脱了`\\`）：

```json
GET /testindex/_search
{
  "query": {
    "query_string" : {
      "query" : "address.\\*: New AND (York OR Jersey)"
    }
  }
}
```

### 提升

每个搜索词生成的子查询是使用[`dis_max` 询问]({{site.url}}{{site.baseurl}}/query-dsl/compound/disjunction-max/) 与`tie_breaker`。为了提高各个字段，请使用`^` 操作员。例如，以下查询提高了`title` 田地为2倍：

```json
GET testindex/_search
{
  "query": {
    "query_string": {
      "fields": [ "title^2", "description" ],
      "query": "wind AND film"
    }
  }
}
```
{% include copy-curl.html %}

为了提升一个领域的所有子场，请在通配符之后指定增强操作员：

```json
GET /testindex/_search
{
  "query": {
    "query_string" : {
      "fields" : ["work_address", "address.*^2"],
      "query" : "New AND (York OR Jersey)"
    }
  }
}
```

### 多个字段搜索的参数

搜索多个字段时，您可以传递其他可选`type` 参数到`query_string` 询问。

范围| 数据类型| 描述
:--- | :--- | :---
`type` | 细绳| 确定OpenSearch如何执行查询并得分结果。有效值是`best_fields`，`bool_prefix`，`most_fields`，`cross_fields`，`phrase`， 和`phrase_prefix`。默认为`best_fields`。有关有效值的描述，请参见[多-匹配查询类型]({{site.url}}{{site.baseurl}}/query-dsl/full-text/multi-match/#multi-match-query-types)。

## 同义词`query_string` 询问

这`query_string` 查询支持多人-术语同义词扩展与`synonym_graph` 令牌过滤器。如果您使用`synonym_graph` 令牌过滤器，OpenSearch创建[匹配短语查询]({{site.url}}{{site.baseurl}}/query-dsl/full-text/match-phrase/) 对于每个同义词。

这`auto_generate_synonyms_phrase_query` 参数指定是否要自动创建匹配短语查询-术语同义词。默认情况下，`auto_generate_synonyms_phrase_query` 是`true`，因此，如果您指定`ml, machine learning` 作为同义词和搜索`ml`，OpenSearch搜索`ml OR "machine learning"`。

另外，您可以匹配多匹马-使用连词的术语同义词。如果您设置`auto_generate_synonyms_phrase_query` 到`false`，OpenSearch搜索`ml OR (machine AND learning)`。

例如，以下查询搜索文本`ml models` 并指定不自动-为每个同义词生成匹配短语查询：

```json
GET /testindex/_search
{
  "query": {
    "query_string": {
      "default_field": "title",
      "query": "ml models",
      "auto_generate_synonyms_phrase_query": false
    }
  }
}
```
{% include copy-curl.html %}

对于此查询，OpenSearch创建以下布尔查询：`(ml OR (machine AND learning)) models`。

## 最低应匹配

这`query_string` 查询将每个操作员周围的查询分开，并为整个输入创建布尔查询。这[`minimum_should_match` 范围]({{site.url}}{{site.baseurl}}/query-dsl/minimum-should-match/) 指定文档必须匹配的最小术语数量必须在搜索结果中返回。例如，以下查询指定`description` 字段必须至少匹配每个搜索结果的两个术语：

```json
GET /testindex/_search
{
  "query": {
    "query_string": {
      "fields": [
        "description"
      ],
      "query": "historical epic film",
      "minimum_should_match": 2
    }
  }
}
```
{% include copy-curl.html %}

对于此查询，OpenSearch创建以下布尔查询：`(description:historical description:epic description:film)~2`。

### 最低应与多个字段匹配

如果在一个中指定了多个字段`query_string` 查询，OpenSearch创建[`dis_max` 询问]({{site.url}}{{site.baseurl}}/query-dsl/compound/disjunction-max/) 对于指定的字段。如果您没有明确指定查询术语的操作员，则整个查询文本被视为一个子句。OpenSearch使用此单一子句为每个字段构建查询。最终的布尔查询包含一个对应于`dis_max` 查询所有字段，因此`minimum_should_match` 参数不应用。

例如，在以下查询中，`historical epic heroic` 被视为一个单一条款：

```json
GET /testindex/_search
{
  "query": {
    "query_string": {
      "fields": [
        "title",
        "description"
      ],
      "query": "historical epic heroic",
      "minimum_should_match": 2
    }
  }
}
```
{% include copy-curl.html %}

对于此查询，OpenSearch创建以下布尔查询：`((title:historical title:epic title:heroic) | (description:historical description:epic description:heroic))`。

如果添加显式操作员（`AND` 或者`OR`）在查询术语中，每个术语被视为一个单独的子句，`minimum_should_match` 可以应用参数。例如，在以下查询中，`historical`，`epic`， 和`heroic` 被认为是单独的子句：

```json
GET /testindex/_search
{
  "query": {
    "query_string": {
      "fields": [
        "title",
        "description"
      ],
      "query": "historical OR epic OR heroic",
      "minimum_should_match": 2
    }
  }
}
```
{% include copy-curl.html %}

对于此查询，OpenSearch创建以下布尔查询：`((title:historical | description:historical) (description:epic | title:epic) (description:heroic | title:heroic))~2`。查询与三个子句中的至少两个匹配。每个条款代表`dis_max` 在两个上查询`title` 和`description` 每个学期的字段。

或者，确保`minimum_should_match` 可以应用，您可以设置`type` 参数为`cross_fields`。这表明在分析输入文本时，应将具有相同分析仪的字段分组在一起：

```json
GET /testindex/_search
{
  "query": {
    "query_string": {
      "fields": [
        "title",
        "description"
      ],
      "query": "historical epic heroic",
      "type": "cross_fields",
      "minimum_should_match": 2
    }
  }
}
```
{% include copy-curl.html %}

对于此查询，OpenSearch创建以下布尔查询：`((title:historical | description:historical) (description:epic | title:epic) (description:heroic | title:heroic))~2`。

但是，如果使用不同的分析仪，则必须在查询中使用明确的操作员，以确保`minimum_should_match` 参数应用于每个项。

## 参数

下表列出了参数`query_string` 查询支持。除所有参数外`query` 是可选的。

范围| 数据类型| 描述
:--- | :--- | :---
`query` | 细绳| 可能包含表达式的文本[查询字符串语法](#query-string-syntax) 用于搜索。必需的。
`allow_leading_wildcard` | 布尔| 指定是否`*` 和`?` 允许作为搜索词的第一个字符。默认为`true`。
`analyze_wildcard` | 布尔| 指定OpenSearch是否应该尝试分析通配符术语。默认为`false`。
`analyzer` | 细绳| 这[分析仪]({{site.url}}{{site.baseurl}}/analyzers/index/) 用于引导查询字符串文本。默认是索引-指定的时间分析仪`default_field`。如果未针对`default_field`， 这`analyzer` 是索引的默认分析仪。
`auto_generate_synonyms_phrase_query` | 布尔| 指定是否创建[匹配短语查询]({{site.url}}{{site.baseurl}}/query-dsl/full-text/match-phrase/) 自动用于多人-术语同义词。例如，如果指定`ba, batting average` 作为同义词和搜索`ba`，OpenSearch搜索`ba OR "batting average"` （如果此选项是`true`） 或者`ba OR (batting AND average)` （如果此选项是`false`）。默认为`true`。
`boost` | 漂浮的-观点| 通过给定的乘数增强子句。对于在复合查询中称量从句有用。[0，1）中的值降低了相关性，并且值大于1的相关性。默认为`1`。
`default_field` | 细绳| 查询字符串中未指定字段的字段。支持通配符。默认为在`index.query. Default_field` 索引设置。默认情况下，`index.query. Default_field` 是`*`，这意味着提取所有有资格的术语查询和过滤元数据字段的字段。如果提取的字段合并为查询`prefix` 未指定。合格的字段不包括嵌套文档。搜索所有合格的字段可能是资源-密集操作。这`indices.query.bool.max_clause_count` 搜索设置定义了字段数的乘积和一次可以查询的条款数的最大值。的默认值`indices.query.bool.max_clause_count` 是1,024。
`default_operator`| 细绳| 如果查询字符串包含多个搜索术语，则所有术语是否需要匹配（`AND`）或只需要一个术语匹配（`OR`）文档被视为匹配项。有效值是：<br>- `OR`：字符串`to be` 被解释为`to OR be`<br>- `AND`：字符串`to be` 被解释为`to AND be`<br>默认值为`OR`。
`enable_position_increments` | 布尔| 什么时候`true`，结果查询知道位置增量。当删除停止单词留下不必要的时，此设置很有用"gap" 在术语之间。默认为`true`。
`fields` | 字符串数组| 要搜索的字段列表（例如，`"fields": ["title^4", "description"]`）。支持通配符。如果未指定，则默认为`index.query. Default_field` 设置，默认为`["*"]`。
`fuzziness` | 细绳| 在确定一个术语是否匹配值时，将一个单词更改为另一个单词所需的字符编辑数量（插入，删除，替换）。例如，`wined` 和`wind` 是1.有效值是非-负整数或`AUTO`。默认，`AUTO`，根据每个学期的长度选择一个值，对于大多数用例，是一个不错的选择。
`fuzzy_max_expansions` | 正整数| 查询可以扩展的最大术语数量。模糊的查询“扩展为”在指定距离内的许多匹配术语`fuzziness`。然后OpenSearch尝试匹配这些术语。默认为`50`。
`fuzzy_transpositions` | 布尔| 环境`fuzzy_transpositions` 到`true` （默认）在插入，删除和替代操作中添加相邻字符的互换`fuzziness` 选项。例如，`wind` 和`wnid` 是1`fuzzy_transpositions` 是真的（交换"n" 和"i"）和2如果是错误的（删除"n"， 插入"n"）。如果`fuzzy_transpositions` 是错误的，`rewind` 和`wnid` 距离有相同的距离（2）`wind`，尽管人类越来越多-以中心的看法`wnid` 是一个明显的错别字。对于大多数用例，默认值是一个不错的选择。
`lenient` | 布尔| 环境`lenient` 到`true` 忽略查询和文档字段之间的数据类型不匹配。例如，一个查询字符串`"8.2"` 可以匹配类型字段`float`。默认为`false`。
`max_determinized_states` | 正整数| 最大数量"[状态](https://lucene.apache.org/core/8_9_0/core/org/apache/lucene/util/automaton/Operations.html#DEFAULT_MAX_DETERMINIZED_STATES)" （一种复杂性的度量）Lucene可以为包含正则表达式的查询字符串创建（例如，`"query": "/wind.+?/"`）。较大的数字允许查询使用更多内存的查询。默认值为10,000。
`minimum_should_match` | 正整数，正数，正百分比，组合| 如果查询字符串包含多个搜索术语，并且您使用`or` 操作员，需要将文档匹配的术语数量被视为匹配。例如，如果`minimum_should_match` 是2，`wind often rising` 不匹配`The Wind Rises.` 如果`minimum_should_match` 是`1`， 它匹配。有关详细信息，请参阅[最低应匹配]({{site.url}}{{site.baseurl}}/query-dsl/minimum-should-match/)。
`phrase_slop` | 整数| 匹配单词之间允许的最大单词数。如果`phrase_slop` IS 2，在短语中匹配的单词之间最多允许两个单词。换词单词的斜率为2。默认值为`0` （必须匹配的确切短语匹配，必须彼此相邻）。
`quote_analyzer` | 细绳| 该分析仪曾经在查询字符串中引用了引用的文本。覆盖`analyzer` 引用文本的参数。默认为`search_quote_analyzer` 为`default_field`。
`quote_field_suffix` | 细绳| 此选项支持使用与非非非分析方法搜索精确匹配（用引号包围）-确切的匹配使用。例如，如果`quote_field_suffix` 是`.exact` 然后您搜索`\"lightly\"` 在里面`title` 字段，openSearch搜索单词`lightly` 在里面`title.exact` 场地。第二个字段可能使用其他类型（例如，`keyword` 而不是`text`）或其他分析仪。
`rewrite` | 细绳| 确定OpenSearch如何重写和分数多数-术语查询。有效值是`constant_score`，`scoring_boolean`，`constant_score_boolean`，`top_terms_N`，`top_terms_boost_N`， 和`top_terms_blended_freqs_N`。默认为`constant_score`。
`time_zone` | 细绳| 指定从`UTC`。如果查询字符串包含日期范围，则需要指示时区偏移号。例如，设置`time_zone": "-08:00"` 对于具有日期范围的查询，例如`"query": "wind rises release_date[2012-01-01 TO 2014-01-01]"`）。用于指定偏移小时数的默认时区格式为`UTC`。

查询字符串查询可以内部转换为[前缀查询]({{site.url}}{{site.baseurl}}/query-dsl/term/prefix/)。如果[`search.allow_expensive_queries`]({{site.url}}{{site.baseurl}}/query-dsl/index/#expensive-queries) 被设定为`false`，前缀查询未执行。如果`index_prefixes` 已启用，`search.allow_expensive_queries` 设置将被忽略，并构建和执行优化的查询。
{： 。重要的


---
layout: default
title: 功能
parent: SQL
grand_parent: SQL和PPL
nav_order: 7
Redirect_from:
  - /search-plugins/sql/functions/
---

# 功能

SQL语言支持所有SQL插件[共同的功能]({{site.url}}{{site.baseurl}}/search-plugins/sql/functions/)， 包括[相关搜索]({{site.url}}{{site.baseurl}}/search-plugins/sql/full-text/)，但还引入了一些函数同义词，仅在SQL中可用。
这些同义词由`V1` 引擎。有关更多信息，请参阅[限制]({{site.url}}{{site.baseurl}}/search-plugins/sql/limitation)。

## 匹配查询

这`MATCHQUERY` 和`MATCH_QUERY` 功能是同义词[`MATCH`]({{site.url}}{{site.baseurl}}/search-plugins/sql/full-text#match) 相关功能。他们不接受其他论点，而是提供替代语法。

### 句法

使用`matchquery` 或者`match_query`，传递您的搜索查询以及您要搜索的字段名称：

```sql
match_query(field_expression, query_expression[, option=<option_value>]*)
matchquery(field_expression, query_expression[, option=<option_value>]*)
field_expression = match_query(query_expression[, option=<option_value>]*)
field_expression = matchquery(query_expression[, option=<option_value>]*)
```

您可以按任何顺序指定以下选项：

- `analyzer`
- `boost`

### 例子

您可以使用`MATCHQUERY` 取代`MATCH`：

```sql
SELECT account_number, address
FROM accounts
WHERE MATCHQUERY(address, 'Holmes')
```

或者，您可以使用`MATCH_QUERY` 取代`MATCH`：

```sql
SELECT account_number, address
FROM accounts
WHERE address = MATCH_QUERY('Holmes')
```

结果包含地址包含的文档"Holmes"：

| 帐号| 地址
:--- | :---
1| 880 Holmes Lane

## 多-匹配

有三个同义词[`MULTI_MATCH`]({{site.url}}{{site.baseurl}}/search-plugins/sql/full-text#multi-match)，每个语法略有不同。他们接受一个查询字符串和具有权重的字段列表。他们还可以接受其他可选参数。

### 句法

```sql
multimatch('query'=query_expression[, 'fields'=field_expression][, option=<option_value>]*)
multi_match('query'=query_expression[, 'fields'=field_expression][, option=<option_value>]*)
multimatchquery('query'=query_expression[, 'fields'=field_expression][, option=<option_value>]*)
```

这`fields` 参数是可选的，可以包含一个字段或逗号-分开的列表（不允许使用Whitespace字符）。每个字段的重量是可选的，并且在字段名称之后指定。应该由`caret` 特点-- `^` -- 没有空格。

### 例子

以下查询显示`fields` 多数参数-使用单个字段和一个字段列表匹配查询：

```sql
multi_match('fields' = "Tags^2,Title^3.4,Body,Comments^0.3", ...)
multi_match('fields' = "Title", ...)
```

您可以按任何顺序指定以下选项：

- `analyzer`
- `boost`
- `slop`
- `type`
- `tie_breaker`
- `operator`

## 请求参数

这`QUERY` 功能是[`QUERY_STRING`]({{site.url}}{{site.baseurl}}/search-plugins/sql/full-text#query-string)。

### 句法

```sql
query('query'=query_expression[, 'fields'=field_expression][, option=<option_value>]*)
```

这`fields` 参数是可选的，可以包含一个字段或逗号-分开的列表（不允许使用Whitespace字符）。每个字段的重量是可选的，并且在字段名称之后指定。应该由`caret` 特点-- `^` -- 没有空格。

### 例子

以下查询显示`fields` 多数参数-使用单个字段和一个字段列表匹配查询：

```sql
query('fields' = "Tags^2,Title^3.4,Body,Comments^0.3", ...)
query('fields' = "Tags", ...)
```

您可以按任何顺序指定以下选项：

- `analyzer`
- `boost`
- `slop`
- `default_field`

### 使用的示例`query_string` 在SQL和PPL查询中：

以下是OpenSearch DSL中的示例REST API搜索请求。

```json
GET accounts/_search
{
  "query": {
    "query_string": {
      "query": "Lane Street",
      "fields": [ "address" ],
    }
  }
}
```

上面的请求等同于以下内容`query` 功能：

```sql
SELECT account_number, address
FROM accounts
WHERE query('address:Lane OR address:Street')
```

结果包含包含的地址"Lane" 或者"Street"：

| 帐号| 地址
:--- | :---
1| 880 Holmes Lane
6| 布里斯托尔街671号
13| 麦迪逊街789号

## 匹配短语

这`MATCHPHRASEQUERY` 功能是[`MATCH_PHRASE`]({{site.url}}{{site.baseurl}}/search-plugins/sql/full-text#query-string)。

### 句法

```sql
matchphrasequery(query_expression, field_expression[, option=<option_value>]*)
```

您可以按任何顺序指定以下选项：

- `analyzer`
- `boost`
- `slop`

## 得分查询

要返回相关得分以及每个匹配文档，请使用`SCORE`，`SCOREQUERY`， 或者`SCORE_QUERY` 功能。

### 句法

这`SCORE` 功能期望两个参数。第一个论点是[`MATCH_QUERY`](#match-query) 表达。第二个论点是可选的浮动-提高分数的点号（默认值为1.0）：

```sql
SCORE(match_query_expression, score)
SCOREQUERY(match_query_expression, score)
SCORE_QUERY(match_query_expression, score)
```

### 例子

以下示例使用`SCORE` 功能以提高文档的分数：

```sql
SELECT account_number, address, _score
FROM accounts
WHERE SCORE(MATCH_QUERY(address, 'Lane'), 0.5) OR
  SCORE(MATCH_QUERY(address, 'Street'), 100)
ORDER BY _score
```

结果包含与相应分数的匹配：

| 帐号| 地址| 分数
:--- | :--- | :---
1| 880 Holmes Lane| 0.5
6| 布里斯托尔街671号| 100
13| 麦迪逊街789号| 100

## 通配符查询

要通过给定的通配符搜索文档，请使用`WILDCARDQUERY` 或者`WILDCARD_QUERY` 功能。

### 句法

```sql
wildcardquery(field_expression, query_expression[, boost=<value>])
wildcard_query(field_expression, query_expression[, boost=<value>])
```

### 例子

以下示例使用通配符查询：

```sql
SELECT account_number, address
FROM accounts
WHERE wildcard_query(address, '*Holmes*');
```

结果包含与通配符表达式相匹配的文档：

| 帐号| 地址
:--- | :---
1| 880 Holmes Lane


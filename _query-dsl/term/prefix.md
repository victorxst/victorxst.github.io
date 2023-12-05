---
layout: default
title: 字首
parent: 术语级查询
grand_parent: 查询DSL
nav_order: 40
---

# 前缀查询

使用`prefix` 查询以搜索以特定前缀开头的术语。例如，以下查询搜索文档`speaker` 字段包含一个以开始的术语`KING H`：

```json
GET shakespeare/_search
{
  "query": {
    "prefix": {
      "speaker": "KING H"
    }
  }
}
```
{% include copy-curl.html %}

要提供参数，您可以使用与前面的查询相同的查询，并使用以下扩展语法：

```json
GET shakespeare/_search
{
  "query": {
    "prefix": {
      "speaker": {
        "value": "KING H"
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
    "prefix": {
      "<field>": {
        "value": "sample",
        ... 
      }
    }
  }
}
```
{% include copy-curl.html %}

这`<field>` 接受以下参数。除所有参数外`value` 是可选的。

范围| 数据类型| 描述
:--- | :--- | :---
`value` | 细绳| 在指定的字段中搜索的术语`<field>`。
`case_insensitive` | 布尔| 如果`true`，允许案例-该值与索引字段值的不敏感匹配。默认为`false` （案例灵敏度由字段的映射确定）。
`rewrite` | 细绳| 确定OpenSearch如何重写和分数多数-术语查询。有效值是`constant_score`，`scoring_boolean`，`constant_score_boolean`，`top_terms_N`，`top_terms_boost_N`， 和`top_terms_blended_freqs_N`。默认为`constant_score`。

如果[`search.allow_expensive_queries`]({{site.url}}{{site.baseurl}}/query-dsl/index/#expensive-queries) 被设定为`false`，前缀查询不运行。如果`index_prefixes` 已启用，`search.allow_expensive_queries` 设置被忽略，并建立并运行优化的查询。
{： 。重要的


---
layout: default
title: 通配符
parent: 术语级查询
grand_parent: 查询DSL
nav_order: 100
---

# 通配符查询

使用通配符查询搜索与通配符模式相匹配的术语。通配符查询支持以下操作员。

操作员| 描述
:--- | :---
`*` | 匹配零或更多字符。
`?` | 匹配任何单个字符。
`case_insensitive` | 如果`true`，通配符查询不敏感。如果`false`，通配符查询对案例敏感。默认为`false` （区分大小写）。

案件-敏感的搜索以开始的术语`H` 并以`Y`，使用以下请求：

```json
GET shakespeare/_search
{
  "query": {
    "wildcard": {
      "speaker": {
        "value": "H*Y",
        "case_insensitive": false
      }
    }
  }
}
```
{% include copy-curl.html %}

如果您更改`*` 到`?`，你没有比赛，因为`?` 指一个字符。

通配符的查询往往很慢，因为它们需要迭代很多术语。避免将通配符的字符放在查询开始时，因为在资源和时间方面，这可能是非常昂贵的操作。

## 参数

查询接受字段的名称（`<field>`）作为顶部-级别参数：

```json
GET _search
{
  "query": {
    "wildcard": {
      "<field>": {
        "value": "patt*rn",
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
`value` | 细绳| 在指定的字段中使用的通配符模式`<field>`。
`boost` | 漂浮的-观点| 通过给定的乘数增强查询。对于包含多个查询的搜索很有用。[0，1）中的值降低了相关性，并且值大于1的相关性。默认为`1`。
`case_insensitive` | 布尔| 如果`true`，允许案例-该值与索引字段值的不敏感匹配。默认为`false` （案例灵敏度由字段的映射确定）。
`rewrite` | 细绳| 确定OpenSearch如何重写和分数多数-术语查询。有效值是`constant_score`，`scoring_boolean`，`constant_score_boolean`，`top_terms_N`，`top_terms_boost_N`， 和`top_terms_blended_freqs_N`。默认为`constant_score`。

如果[`search.allow_expensive_queries`]({{site.url}}{{site.baseurl}}/query-dsl/index/#expensive-queries) 被设定为`false`，通配符查询不运行。
{: .important}


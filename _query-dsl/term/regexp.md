---
layout: default
title: Regexp
parent: 术语级查询
grand_parent: 查询DSL
nav_order: 60
---

# Regexp查询

使用`regexp` 查询以搜索与正则表达式匹配的术语。

以下查询搜索任何以任何大写或小写字母开头的术语`amlet`：

```json
GET shakespeare/_search
{
  "query": {
    "regexp": {
      "play_name": "[a-zA-Z]amlet"
    }
  }
}
```
{% include copy-curl.html %}

请注意以下重要考虑因素：

- 正则表达式应用于现场的术语（即令牌）---不去整个领域。
- 默认情况下，正则表达式的最大长度为1,000个字符。要更改最大长度，请更新`index.max_regex_length` 环境。
- 正则表达式使用Lucene语法，该语法与更标准化的实现不同。彻底测试以确保您收到期望的结果。要了解更多，请参阅[Lucene文档](https://lucene.apache.org/core/8_9_0/core/index.html)。
- 为了提高REGEXP查询性能，请避免使用前缀或后缀的通配符图案，例如`.*` 或者`.*?+`。
- `regexp` 查询可能是昂贵的操作，需要[`search.allow_expensive_queries`]({{site.url}}{{site.baseurl}}/query-dsl/index/#expensive-queries) 设置为`true`。在频繁之前`regexp` 查询，测试其对聚类性能的影响，并检查可能获得相似结果的替代查询。

## 参数

查询接受字段的名称（`<field>`）作为顶部-级别参数：

```json
GET _search
{
  "query": {
    "regexp": {
      "<field>": {
        "value": "[Ss]ample",
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
`value` | 细绳| 在指定的字段中使用的正则表达式`<field>`。
`case_insensitive` | 布尔| 如果`true`，允许案例-正则表达值与索引场值的不敏感匹配。默认为`false` （案例灵敏度由字段的映射确定）。
`flags` | 细绳| 为Lucene的正则表达引擎启用可选操作员。
`max_determinized_states` | 整数| Lucene将正则表达式转换为具有许多确定状态的自动机。此参数指定查询所需的最大自动机状态。使用此参数防止高资源消耗。要运行复杂的正则表达式，您可能需要增加此参数的值。默认值为10,000。
`rewrite` | 细绳| 确定OpenSearch如何重写和分数多数-术语查询。有效值是`constant_score`，`scoring_boolean`，`constant_score_boolean`，`top_terms_N`，`top_terms_boost_N`， 和`top_terms_blended_freqs_N`。默认为`constant_score`。

如果[`search.allow_expensive_queries`]({{site.url}}{{site.baseurl}}/query-dsl/index/#expensive-queries) 被设定为`false`，`regexp` 查询不运行。
{： 。重要的}



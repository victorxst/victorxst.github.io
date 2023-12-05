---
layout: default
title: 模糊
parent: 术语级查询
grand_parent: 查询DSL
nav_order: 20
---

# 模糊查询

模糊查询搜索文档，其中包含与搜索词相似的术语的文档[Levenshtein距离](https://en.wikipedia.org/wiki/Levenshtein_distance)。Levenshtein距离测量一个-将一个术语更改为另一个术语所需的角色更改。这些更改包括：

- 替换：**C**在**b**在
- 插入：猫猫**s**
- 删除：**C**at at
- 换位：**CA**t**交流**t

模糊查询创建了列出Levenshtein距离内的所有可能扩展词的列表。您可以指定此类扩展的最大数量`max_expansions` 场地。然后，它搜索与任何扩展相匹配的文档。

以下示例查询搜索扬声器`HALET` （拼写错误`HAMLET`）。未指定最大编辑距离，因此默认`AUTO` 使用编辑距离：

```json
GET shakespeare/_search
{
  "query": {
    "fuzzy": {
      "speaker": {
        "value": "HALET"
      }
    }
  }
}
```
{％包含副本-curl.html％}

响应包含所有文件`HAMLET` 是演讲者。

以下示例查询搜索单词`HALET` 使用高级参数：

```json
GET shakespeare/_search
{
  "query": {
    "fuzzy": {
      "speaker": {
        "value": "HALET",
        "fuzziness": "2",
        "max_expansions": 40,
        "prefix_length": 0,
        "transpositions": true,
        "rewrite": "constant_score"
      }
    }
  }
}
```
{％包含副本-curl.html％}

## 参数

查询接受字段的名称（`<field>`）作为顶部-级别参数：

```json
GET _search
{
  "query": {
    "fuzzy": {
      "<field>": {
        "value": "sample",
        ... 
      }
    }
  }
}
```
{％包含副本-curl.html％}

这`<field>` 接受以下参数。除所有参数外`value` 是可选的。

范围| 数据类型| 描述
：--- | ：--- | ：---
`value` | 细绳| 在指定的字段中搜索的术语`<field>`。
`fuzziness` | `AUTO`，，，，`0`，或一个积极的整数| 当确定一个术语是否匹配值时，需要将一个单词更改为另一个单词的字符编辑数量（插入，删除，替代）。例如，`wined` 和`wind` 是1.默认`AUTO`，根据每个学期的长度选择一个值，对于大多数用例，是一个不错的选择。
`max_expansions` | 正整数|  查询可以扩展的最大术语数量。模糊的查询“扩展为”在指定距离内的许多匹配术语`fuzziness`。然后OpenSearch尝试匹配这些术语。默认为`50`。
`prefix_length` | 非-负整数| 在模糊性中未考虑的领先角色的数量。默认为`0`。
`rewrite` | 细绳| 确定OpenSearch如何重写和分数多数-术语查询。有效值是`constant_score`，，，，`scoring_boolean`，，，，`constant_score_boolean`，，，，`top_terms_N`，，，，`top_terms_boost_N`， 和`top_terms_blended_freqs_N`。默认为`constant_score`。
`transpositions` | 布尔| 指定是否允许两个相邻字符的换位（`ab` 到`ba`）作为编辑。默认为`true`。

指定很大的价值`max_expansions` 会导致性能差，尤其是`prefix_length` 被设定为`0`，由于开启搜索试图匹配的单词的变化很大。
{： 。警告}

如果[`search.allow_expensive_queries`]({{site.url}}{{site.baseurl}}/query-dsl/index/#expensive-queries) 被设定为`false`，不运行模糊查询。
{： 。重要的


---
layout: default
title: 关键词
nav_order: 46
has_children: false
parent: String字段类型
grand_parent: 支持的字段类型
redirect_from:
  - /opensearch/supported-field-types/keyword/
  - /field-types/keyword/
---

# 关键字字段类型

关键字字段类型包含未分析的字符串。它仅允许精确的情况-敏感匹配。

如果您需要使用一个字段以完整-文本搜索，将其映射为[`text`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/text/) 反而。
{: .note}

## 例子

使用关键字字段创建映射：

```json
PUT movies
{
  "mappings" : {
    "properties" : {
      "genre" : {
        "type" :  "keyword"
      }
    }
  }
}
```
{% include copy-curl.html %}

## 参数

下表列出了由关键字字段类型接受的参数。所有参数都是可选的。

范围| 描述
:--- | :--- 
`boost` | 浮动-指定该字段对相关性分数的重量的点值。值高于1.0的值增加了该领域的相关性。0.0至1.0之间的值降低了该场的相关性。默认值为1.0。
`doc_values` | 布尔值指定是否应将字段存储在磁盘上，以便将其用于聚合，排序或脚本。默认为`false`。
`eager_global_ordinals` | 指定是否应在刷新上热切地加载全球序列。如果该字段通常用于聚合，则应将此参数设置为`true`。默认为`false`。
`fields` | 要以多种方式索引相同的字符串（例如，作为关键字和文本），提供字段参数。您可以指定用于搜索的字段的一个版本，而用于分类和聚合的另一个版本。
`ignore_above` | 任何更长的字符串都不应索引。默认值为2147483647。默认动态映射创建一个关键字子字段。`ignore_above` 设置为256。
`index` | 布尔值指定是否应搜索该字段。默认为`true`。
`index_options` | 在计算相关性分数时将考虑的索引中存储的信息。可以设置为`freqs` 用于期限频率。默认为`docs`。
`meta` | 接受该领域的元数据。
`normalizer` | 指定在索引之前如何预处理此字段（例如，使其成为小写）。默认为`null` （没有预处理）。
`norms` | 布尔值指定在计算相关性分数时是否应使用字段长度。默认为`false`。
[`null_value`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/index#null-value) | 用于代替的值`null`。必须与字段相同。如果未指定此参数，则该字段在其值为时被视为丢失`null`。默认为`null`。
`similarity` | 用于计算相关性得分的排名算法。默认为`BM25`。
`split_queries_on_whitespace` | 布尔值，指定是否满-文本查询应在空格上分开。默认为`false`。
`store` | 布尔值指定是否应存储字段值，并且可以与_source字段分开检索。默认为`false`。


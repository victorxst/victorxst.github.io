---
layout: default
title: 文本
nav_order: 47
has_children: false
parent: String字段类型
grand_grand_parent: 支持的字段类型
redirect_from:
  - /opensearch/supported-field-types/text/
  - /field-types/text/
---

# 文本字段类型

文本字段类型包含一个分析的字符串。它用于完整-文本搜索是因为它允许部分匹配。具有多个术语的搜索可以匹配一些但不匹配所有术语。根据分析仪的不同，结果可能是病例不敏感的，STEM的，删除了止动，施加了同义词，等等。


如果您需要使用一个字段以确切-价值搜索，将其映射为[`keyword`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/keyword/) 反而。
{:.note}

## 例子

使用文本字段创建映射：

```json
PUT movies
{
  "mappings" : {
    "properties" : {
      "title" : {
        "type" :  "text"
      }
    }
  }
}
```
{% include copy-curl.html %}

## 参数

下表列出了文本字段类型接受的参数。所有参数都是可选的。

范围| 描述
:--- | :---
`analyzer` | 用于此字段的分析仪。默认情况下，它将在索引时和搜索时间使用。要在搜索时间覆盖它，请设置`search_analyzer` 范围。默认为`standard` 分析仪，使用语法-基于令牌化，并基于[Unicode文本细分](https://unicode.org/reports/tr29/) 算法。
`boost` | 浮动-指定该字段对相关性分数的重量的点值。值高于1.0的值增加了该领域的相关性。0.0至1.0之间的值降低了该场的相关性。默认值为1.0。
`eager_global_ordinals` | 指定是否应在刷新上热切地加载全球序列。如果该字段通常用于聚合，则应将此参数设置为`true`。默认为`false`。
`fielddata` | 布尔值指定是否访问该字段的代币进行分析，汇总和脚本。默认为`false`。
`fielddata_frequency_filter` | 一个指定仅加载到存储器的JSON对象`min` 和`max` 值（作为绝对数字或百分比提供）。频率是每个段计算的。参数：`min`，`max`，`min_segment_size`。默认值是加载所有分析的令牌。
`fields` | 要以多种方式索引相同的字符串（例如，作为关键字和文本），提供字段参数。您可以指定用于搜索的字段的一个版本，而用于分类和聚合的另一个版本。
`index` | 布尔值指定是否应搜索该字段。默认为`true`。
`index_options` | 指定要存储在索引中的信息进行搜索和突出显示。有效值：`docs` （仅DOC编号），`freqs` （文档编号和术语频率），`positions` （文档编号，任期频率和期限位置），`offsets` （文档编号，术语频率，术语位置以及启动和最终字符偏移）。默认为`positions`。
`index_phrases` | 指定索引2的布尔值-克分别。2-克是该字段字符串中两个连续单词的组合。导致更快的精确短语查询，没有斜率，而是较大的索引。当未删除停止词时，效果最好。默认为`false`。
`index_prefixes` | 一个分别指定索引项前缀的JSON对象。前缀中的字符数在`min_chars` 和`max_chars`， 包括的。导致更快的前缀搜索，但索引较大。可选参数：`min_chars`，`max_chars`。默认`min_chars` 是2，`max_chars` 是5。
`meta` | 接受该领域的元数据。
`norms` | 布尔值指定在计算相关性分数时是否应使用字段长度。默认为`false`。
`position_increment_gap` | 分析文本字段时，将分配它们的位置。如果一个字段包含一系列字符串，并且这些位置是连续的，则将导致在不同的阵列元素上可能匹配。为了防止这种情况，在连续的阵列元素之间插入人造间隙。您可以通过指定整数来更改此差距`position_increment_gap`。注意：如果`slop` 大于`position_element_gap`，可能会发生在不同的数组元素上匹配。默认值为100。
`similarity` | 用于计算相关性得分的排名算法。默认为`BM25`。
[`term_vector`](#term-vector-parameter) | 布尔值指定是否应存储该字段的术语向量。默认为`no`。

## 项矢量参数

分析过程中产生一个术语矢量。它包含了：
- 术语列表。
- 每个项的序列位置。
- 字段中搜索字符串的开始和最终字符偏移。
- 有效载荷（如果有）。每个项可以具有与该术语位置关联的自定义二进制数据。

这`term_vector` 字段包含一个接受以下参数的JSON对象：

范围| 存储的值
:--- | :---
`no` | 没有任何。这是默认值。
`yes` | 字段中的术语。
`with_offsets` | 条款和性格偏移。
`with_positions_offsets` | 术语，位置和角色偏移。
`with_positions_offsets_payloads` | 术语，位置，角色偏移和有效载荷。
`with_positions` | 条款和立场。
`with_positions_payloads` | 术语，职位和有效载荷。

存储位置可用于接近查询。存储字符偏移可用于突出显示。
{: .tip }

### 术语矢量参数示例

使用文本字段创建一个映射，该文本字段将字符偏移存储在术语向量中：

```json
PUT testindex
{
  "mappings" : {
    "properties" : {
      "dob" : {
        "type" :  "text",
        "term_vector": "with_positions_offsets"
      }
    }
  }
}
```
{% include copy-curl.html %}

带有文本字段的文档索引：

```json
PUT testindex/_doc/1
{
    "dob" : "The patient's date of birth."
}
```
{% include copy-curl.html %}

查询"date of birth" 并在原始字段中突出显示：

```json
GET testindex/_search
{
  "query": {
    "match": {
      "text": "date of birth"
    }
  },
  "highlight": {
    "fields": {
      "text": {} 
    }
  }
}
```
{% include copy-curl.html %}

话"date of birth" 在回应中强调：

```json
{
  "took" : 854,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.8630463,
    "hits" : [
      {
        "_index" : "testindex",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.8630463,
        "_source" : {
          "text" : "The patient's date of birth."
        },
        "highlight" : {
          "text" : [
            "The patient's <em>date</em> <em>of</em> <em>birth</em>."
          ]
        }
      }
    ]
  }
}
```

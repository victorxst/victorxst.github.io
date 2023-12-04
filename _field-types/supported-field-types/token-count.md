---
layout: default
title: 令牌计数
nav_order: 48
has_children: false
parent: String字段类型
grand_grand_parent: 支持的字段类型
redirect_from:
  - /opensearch/supported-field-types/token-count/
  - /field-types/token-count/
---

# 令牌计数字段类型

代币计数字段类型将字符串中分析令牌的数量存储。

## 例子

使用令牌计数字段创建映射：

```json
PUT testindex
{
  "mappings": {
    "properties": {
      "sentence": { 
        "type": "text",
        "fields": {
          "num_words": { 
            "type":     "token_count",
            "analyzer": "english"
          }
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

索引带有文本字段的三个文档：

```json
PUT testindex/_doc/1
{ "sentence": "To be, or not to be: that is the question." }
```
{% include copy-curl.html %}

```json
PUT testindex/_doc/2
{ "sentence": "All the world’s a stage, and all the men and women are merely players." }
```
{% include copy-curl.html %}

```json
PUT testindex/_doc/3
{ "sentence": "Now is the winter of our discontent." }
```
{% include copy-curl.html %}

搜索少于10个单词的句子：

```json
GET testindex/_search
{
  "query": {
    "range": {
      "sentence.num_words": {
        "lt": 10
      }
    }
  }
}
```
{% include copy-curl.html %}

响应包含一个匹配句子：

```json
{
  "took" : 8,
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
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "testindex",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "sentence" : "Now is the winter of our discontent."
        }
      }
    ]
  }
}
```

## 参数

下表列出了由令牌计数字段类型接受的参数。这`analyzer` 需要参数；所有其他参数都是可选的。

范围| 描述
：--- | ：--- 
`analyzer` | 用于此字段的分析仪。指定无令牌过滤器的分析仪以获得最佳性能。必需的。
`boost` | 浮动-指定该字段对相关性分数的重量的点值。值高于1.0的值增加了该领域的相关性。0.0至1.0之间的值降低了该场的相关性。默认值为1.0。
`doc_values` | 布尔值指定是否应将字段存储在磁盘上，以便将其用于聚合，排序或脚本。默认为`false`。
`enable_position_increments` | 布尔值指定是否应计数位置增量。为避免删除停止词，请将此字段设置为`false`。默认为`true`。
`index` | 布尔值指定是否应搜索该字段。默认为`true`。
[`null_value`]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/index#null-value) | 用于代替的值`null`。必须与字段相同。如果未指定此参数，则该字段在其值为时被视为丢失`null`。默认为`null`。
`store` | 布尔值指定是否应存储字段值，并且可以与_source字段分开检索。默认为`false`。


---
layout: default
title: 查询和过滤上下文
nav_order: 5
redirect_from:
- /query-dsl/query-dsl/query-filter-context/
---

# 查询和过滤上下文

查询由查询子句组成，可以在[_ filter context _](#filter-context) 或者[_Query Context_](#query-context)。过滤器上下文中的查询子句问一个问题"_Does_ the document match the query clause?" 并返回匹配文档。查询上下文中的查询子句问一个问题"_How well_ does the document match the query clause?"，返回匹配文档，并以a的形式提供每个文档的相关性[_relevance得分_](#relevance-score)。

## 相关得分

_relevance分数_衡量文档与查询的匹配程度。这是一个积极的浮动-打开搜索记录的点号`_score` 每个文档的元数据字段：

```json
"hits" : [
      {
        "_index" : "shakespeare",
        "_id" : "32437",
        "_score" : 18.781435,
        "_source" : {
          "type" : "line",
          "line_id" : 32438,
          "play_name" : "Hamlet",
          "speech_number" : 3,
          "line_number" : "1.1.3",
          "speaker" : "BERNARDO",
          "text_entry" : "Long live the king!"
        }
      },
...
```

更高的分数表示更相关的文档。尽管不同的查询类型的计算相关性分数不同，但所有查询类型都考虑到在过滤器还是查询上下文中运行查询子句。

使用要在查询上下文中影响相关性分数的查询子句，并在过滤器上下文中使用所有其他查询子句。
{:.tip}

## 滤波器上下文

过滤器上下文中的查询子句问一个问题"_Does_ the document match the query clause?"，有二进制答案。例如，如果您有带有学生数据的索引，则可以使用过滤器上下文回答有关学生的以下问题：

- 是学生的`honors` 状态设置为`true`？
- 是学生的`graduation_year` 在2020年--2022范围？

使用过滤器上下文，OpenSearch返回匹配文档，而无需计算相关得分。因此，您应该使用具有精确值的字段使用过滤器上下文。

要在滤波器上下文中运行查询子句，请将其传递给`filter` 范围。例如，以下布尔查询搜索2020年以荣誉毕业的学生--2022：

```json
GET students/_search
{
  "query": { 
    "bool": { 
      "filter": [ 
        { "term":  { "honors": true }},
        { "range": { "graduation_year": { "gte": 2020, "lte": 2022 }}}
      ]
    }
  }
}
```

为了提高性能，OpenSearch Caches经常使用过滤器。

## 查询上下文

查询上下文中的查询子句问一个问题"_How well_ does the document match the query clause?"，没有二进制答案。查询环境适合完整-文本搜索，您不仅要接收匹配文档，还要确定每个文档的相关性。例如，如果您的索引具有莎士比亚的完整作品，则可以使用查询上下文进行以下搜索：

- 查找包含单词的文档`dream`，包括各种形式（`dreaming` 或者`dreams`）和同义词（`contemplate`）。
- 查找与单词相匹配的文档`long live king`。

在查询上下文中，每个匹配文档都包含相关得分`_score` 您可以使用的字段[种类]({{site.url}}{{site.baseurl}}/opensearch/search/sort) 通过相关性。

要在查询上下文中运行查询子句，请将其传递给`query` 范围。例如，以下查询搜索与单词匹配的文档`long live king` 在里面`shakespeare` 指数：

```json
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": "long live king"
    }
  }
}
```

相关得分是单个-精密浮动-点数24-钻头显着的精度。如果分数计算超过显着的精度，可能会发生精度损失。
{： 。笔记


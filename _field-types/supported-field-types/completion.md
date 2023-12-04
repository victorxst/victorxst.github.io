---
layout: default
title: 完成
nav_order: 51
has_children: false
parent: 自动完成字段类型
grand_parent: 支持的字段类型
redirect_from:
  - /opensearch/supported-field-types/completion/
  - /field-types/completion/
---

# 完成字段类型

完成字段类型通过完成建议提供自动完成功能。完成建议是前缀Suggester，因此仅与文本的开头相匹配。一个完整的建议创建了一个-内存数据结构，可提供更快的查找，但会导致内存使用增加。在使用此功能之前，您需要将所有可能完成的列表上传到索引中。

## 例子

创建一个具有完成字段的映射：

```json
PUT chess_store
{
  "mappings": {
    "properties": {
      "suggestions": {
        "type": "completion"
      },
      "product": {
        "type": "keyword"
      }
    }
  }
}
```
{% include copy-curl.html %}

索引建议：

```json
PUT chess_store/_doc/1
{
  "suggestions": {
      "input": ["Books on openings", "Books on endgames"],
      "weight" : 10
    }
}
```
{% include copy-curl.html %}

## 参数

下表列出了由完成字段接受的参数。

范围| 描述
：--- | ：--- 
`input` | 可能的完成列表，作为字符串或字符串数​​组。无法包含`\u0000` （无效的），`\u001f` （信息分离器一个）或`\u001e` （信息分离器二）。必需的。
`weight` | 积极整数或用于对建议进行排名的正整数。选修的。

可以将多个建议索引如下：

```json
PUT chess_store/_doc/2
{
  "suggestions": [
    {
      "input": "Chess set",
      "weight": 20
    },
    {
      "input": "Chess pieces",
      "weight": 10
    },
    {
      "input": "Chess board",
      "weight": 5
    }
  ]
}
```
{% include copy-curl.html %}

作为替代方案，您可以使用以下速记符号（请注意，您无法提供`weight` 该符号中的参数）：

```json
PUT chess_store/_doc/3
{
  "suggestions" : [ "Chess clock", "Chess timer" ]
}
```
{% include copy-curl.html %}

## 查询完成字段类型

要查询完成字段类型，请指定要搜索的前缀以及寻找建议的字段名称。

查询索引是否以单词开头的建议"chess"：

```json
GET chess_store/_search
{
  "suggest": {
    "product-suggestions": {
      "prefix": "chess",        
      "completion": {         
          "field": "suggestions"
      }
    }
  }
}
```
{% include copy-curl.html %}

响应包含自动完整的建议：

```json
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "suggest" : {
    "product-suggestions" : [
      {
        "text" : "chess",
        "offset" : 0,
        "length" : 5,
        "options" : [
          {
            "text" : "Chess set",
            "_index" : "chess_store",
            "_type" : "_doc",
            "_id" : "2",
            "_score" : 20.0,
            "_source" : {
              "suggestions" : [
                {
                  "input" : "Chess set",
                  "weight" : 20
                },
                {
                  "input" : "Chess pieces",
                  "weight" : 10
                },
                {
                  "input" : "Chess board",
                  "weight" : 5
                }
              ]
            }
          },
          {
            "text" : "Chess clock",
            "_index" : "chess_store",
            "_type" : "_doc",
            "_id" : "3",
            "_score" : 1.0,
            "_source" : {
              "suggestions" : [
                "Chess clock",
                "Chess timer"
              ]
            }
          }
        ]
      }
    ]
  }
}
```

在回应中`_score` 字段包含`weight` 在索引时间设置的参数。这`text` 领域填充了建议的`input` 范围。

默认情况下，响应包含整个文档，包括`_source` 场，可能会影响性能。仅返回`suggestions` 字段，您可以在`_source` 范围。您还可以通过指定返回建议的数量`size` 范围。

```json
GET chess_store/_search
{
  "_source": "suggestions", 
  "suggest": {
    "product-suggestions": {
      "prefix": "chess",        
      "completion": {         
          "field": "suggestions",
          "size" : 3
      }
    }
  }
}
```
{% include copy-curl.html %}

响应包含建议：

```json
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "suggest" : {
    "product-suggestions" : [
      {
        "text" : "chess",
        "offset" : 0,
        "length" : 5,
        "options" : [
          {
            "text" : "Chess set",
            "_index" : "chess_store",
            "_type" : "_doc",
            "_id" : "2",
            "_score" : 20.0,
            "_source" : {
              "suggestions" : [
                {
                  "input" : "Chess set",
                  "weight" : 20
                },
                {
                  "input" : "Chess pieces",
                  "weight" : 10
                },
                {
                  "input" : "Chess board",
                  "weight" : 5
                }
              ]
            }
          },
          {
            "text" : "Chess clock",
            "_index" : "chess_store",
            "_type" : "_doc",
            "_id" : "3",
            "_score" : 1.0,
            "_source" : {
              "suggestions" : [
                "Chess clock",
                "Chess timer"
              ]
            }
          }
        ]
      }
    ]
  }
}
```

要利用源过滤，请在`_search` 端点。这`_suggest` 端点不支持源过滤。
{： 。笔记}

## 完成查询参数

下表列出了完整建议询问的参数。

范围| 描述
：--- | ：--- 
`field` | 一个指定要运行查询的字段的字符串。必需的。
`size` | 一个整数指定最大返回建议的数量。选修的。默认值为5。
`skip_duplicates` | 布尔值指定是否跳过重复建议。选修的。默认为`false`。

## 模糊完成查询

为了允许模糊匹配，您可以指定`fuzziness` 完成查询的参数。在这种情况下，即使用户误解了搜索词，完成查询仍然会返回结果。此外，匹配查询的前缀越长，文档的分数就越高。

```json
GET chess_store/_search
{
  "suggest": {
    "product-suggestions": {
      "prefix": "chesc",        
      "completion": {         
          "field": "suggestions",
          "size" : 3,
          "fuzzy" : {
            "fuzziness" : "AUTO"
          }
      }
    }
  }
}
```
{% include copy-curl.html %}

要使用所有默认模糊选项，请指定`"fuzzy": {}` 或者`"fuzzy": true`。
{： 。提示}

下表列出了模糊完成建议的参数。所有参数都是可选的。

范围| 描述
：--- | ：--- 
`fuzziness` | 模糊性可以设置为以下一个：<br> 1.指定最大允许的整数[Levenshtein距离](https://en.wikipedia.org/wiki/Levenshtein_distance) 对于此编辑。 <br> 2。`AUTO`：0-2个字符的字符串必须完全匹配，3-5个字符的字符串允许1个编辑，字符串超过5个字符允许2个编辑。<br>默认值为`AUTO`。
`min_length` | 一个整数指定输入必须是开始返回建议的最小长度。如果搜索词比`min_length`，没有任何建议。默认值为3。
`prefix_length` | 指定最小长度的整数必须是开始返回建议。如果前缀`prefix_length` 不匹配，但是搜索词仍在Levenshtein距离之内，没有返回建议。默认值为1。
`transpositions` | 一个布尔值指定的是将换位（相邻字符的互换）视为一个编辑而不是两个。示例：建议的`input` 参数为`abcde` 和`fuzziness` 是1.如果`transpositions` 被设定为`true`，，，，`abdce` 会匹配，但是如果`transpositions` 被设定为`false`，，，，`abdce` 不匹配。默认为`true`。
`unicode_aware` | 布尔值指定在测量编辑距离，换位和长度时是否使用Unicode代码点。如果`unicode_aware` 被设定为`true`，测量较慢。默认为`false`，在这种情况下，距离以字节为单位。

## 正则查询

您可以使用正则表达式来定义完整建议查询的前缀。

例如，搜索以"a" 并有一个"d" 稍后，使用以下查询：

```json
GET chess_store/_search
{
  "suggest": {
    "product-suggestions": {
      "regex": "a.*d",        
      "completion": {         
          "field": "suggestions"
      }
    }
  }
}
```
{% include copy-curl.html %}

响应与字符串匹配"abcde"：

```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "suggest" : {
    "product-suggestions" : [
      {
        "text" : "a.*d",
        "offset" : 0,
        "length" : 4,
        "options" : [
          {
            "text" : "abcde",
            "_index" : "chess_store",
            "_type" : "_doc",
            "_id" : "2",
            "_score" : 20.0,
            "_source" : {
              "suggestions" : [
                {
                  "input" : "abcde",
                  "weight" : 20
                }
              ]
            }
          }
        ]
      }
    ]
  }
}
```

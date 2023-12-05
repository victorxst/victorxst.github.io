---
layout: default
title: Boolean
parent: 复合查询
grand_parent: 查询DSL
nav_order: 10
redirect_from:
  - /opensearch/query-dsl/compound/bool/
  - /opensearch/query-dsl/bool/
  - /query-dsl/query-dsl/compound/bool/
---

# 布尔查询

布尔人（`bool`）查询可以将几个查询子句组合到一个高级查询中。该子句与布尔逻辑相结合，以查找结果中返回的匹配文档。

在一个内使用以下查询子句`bool` 询问：

条款| 行为
：--- | ：---
`must` | 逻辑`and` 操作员。结果必须匹配此条款中的所有查询。
`must_not` | 逻辑`not` 操作员。所有匹配都排除在结果之外。
`should` | 逻辑`or` 操作员。结果必须匹配至少一个查询。匹配更多`should` 条款增加了文档的相关性分数。您可以设置必须使用该查询数量的最小数量[`minimum_should_match`]({{site.url}}{{site.baseurl}}/query-dsl/query-dsl/minimum-should-match/) 范围。如果查询包含一个`must` 或者`filter` 条款，默认`minimum_should_match` 值为0。否则，默认`minimum_should_match` 值为1。
`filter` | 逻辑`and` 在应用查询之前首先应用以减少数据集的操作员。过滤器子句中的查询是是或否选项。如果文档匹配查询，则将其返回结果；否则，不是。通常将过滤器查询的结果缓存，以允许更快的返回。使用过滤器查询根据精确匹配，范围，日期或数字过滤结果。

布尔查询具有以下结构：

```json
GET _search
{
  "query": {
    "bool": {
      "must": [
        {}
      ],
      "must_not": [
        {}
      ],
      "should": [
        {}
      ],
      "filter": {}
    }
  }
}
```

例如，假设您在OpenSearch集群中具有莎士比亚索引的完整作品。您想构建一个符合以下要求的单个查询：

1. 这`text_entry` 字段必须包含单词`love` 并且应该包含`life` 或者`grace`。
2. 这`speaker` 字段不得包含`ROMEO`。
3. 将这些结果过滤到戏剧`Romeo and Juliet` 不影响相关得分。

这些要求可以在以下查询中合并：

```json
GET shakespeare/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "text_entry": "love"
          }
        }
      ],
      "should": [
        {
          "match": {
            "text_entry": "life"
          }
        },
        {
          "match": {
            "text_entry": "grace"
          }
        }
      ],
      "minimum_should_match": 1,
      "must_not": [
        {
          "match": {
            "speaker": "ROMEO"
          }
        }
      ],
      "filter": {
        "term": {
          "play_name": "Romeo and Juliet"
        }
      }
    }
  }
}
```

响应包含匹配文档：

```json
{
  "took": 12,
  "timed_out": false,
  "_shards": {
    "total": 4,
    "successful": 4,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 11.356054,
    "hits": [
      {
        "_index": "shakespeare",
        "_id": "88020",
        "_score": 11.356054,
        "_source": {
          "type": "line",
          "line_id": 88021,
          "play_name": "Romeo and Juliet",
          "speech_number": 19,
          "line_number": "4.5.61",
          "speaker": "PARIS",
          "text_entry": "O love! O life! not life, but love in death!"
        }
      }
    ]
  }
}
```

如果要识别以下哪个子句实际引起匹配结果，请用`_name` 范围。
添加`_name` 参数，更改字段名称`match` 查询对象：

```json
GET shakespeare/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "text_entry": {
              "query": "love",
              "_name": "love-must"
            }
          }
        }
      ],
      "should": [
        {
          "match": {
            "text_entry": {
              "query": "life",
              "_name": "life-should"
            }
          }
        },
        {
          "match": {
            "text_entry": {
              "query": "grace",
              "_name": "grace-should"
            }
          }
        }
      ],
      "minimum_should_match": 1,
      "must_not": [
        {
          "match": {
            "speaker": {
              "query": "ROMEO",
              "_name": "ROMEO-must-not"
            }
          }
        }
      ],
      "filter": {
        "term": {
          "play_name": "Romeo and Juliet"
        }
      }
    }
  }
}
```

OpenSearch返回a`matched_queries` 列出与这些结果相匹配的查询的数组：

```json
"matched_queries": [
  "love-must",
  "life-should"
]
```

如果您删除此列表中的查询，则仍然会看到完全相同的结果。
通过检查哪个`should` 条款匹配，您可以更好地了解结果的相关性得分。

您也可以通过嵌套来构建复杂的布尔表达式`bool` 查询。
例如，使用以下查询查找`text_entry` 匹配的字段（`love` 或者`hate`） 和 （`life` 或者`grace`）在剧中`Romeo and Juliet`：

```json
GET shakespeare/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "bool": {
            "should": [
              {
                "match": {
                  "text_entry": "love"
                }
              },
              {
                "match": {
                  "text": "hate"
                }
              }
            ]
          }
        },
        {
          "bool": {
            "should": [
              {
                "match": {
                  "text_entry": "life"
                }
              },
              {
                "match": {
                  "text": "grace"
                }
              }
            ]
          }
        }
      ],
      "filter": {
        "term": {
          "play_name": "Romeo and Juliet"
        }
      }
    }
  }
}
```

响应包含匹配文档：

```json
{
  "took": 10,
  "timed_out": false,
  "_shards": {
    "total": 2,
    "successful": 2,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 11.37006,
    "hits": [
      {
        "_index": "shakespeare",
        "_type": "doc",
        "_id": "88020",
        "_score": 11.37006,
        "_source": {
          "type": "line",
          "line_id": 88021,
          "play_name": "Romeo and Juliet",
          "speech_number": 19,
          "line_number": "4.5.61",
          "speaker": "PARIS",
          "text_entry": "O love! O life! not life, but love in death!"
        }
      }
    ]
  }
}
```


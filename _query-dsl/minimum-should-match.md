---
layout: default
title: 最小值应匹配
nav_order: 70
redirect_from:
- /query-dsl/query-dsl/minimum-should-match/
---

# 最小值应匹配


这`minimum_should_match` 参数可用于完整-文本搜索并指定必须匹配文档的最小术语数量，以便在搜索结果中返回。

以下示例需要一个文档匹配三个搜索字词中的至少两个，以作为搜索结果返回：

```json
GET /shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": {
        "query": "prince king star",
        "minimum_should_match": "2"
      }
    }
  }
}
```

在此示例中，查询具有三个可选子句，它们与`OR`，因此文档必须匹配`prince`，`king`， 或者`star`。

## 有效值

您可以指定`minimum_should_match` 参数是以下值之一。

值类型| 例子| 描述
:--- | :--- | :---
非-负整数| `2` | 文档必须匹配此数量的可选子句。
负整数| `-1` | 文档必须匹配可选子句的总数减去此数字。
非-负百分比| `70%` | 文档必须匹配可选子句总数的这一百分比。匹配的子句数量被舍入到最近的整数。
负百分比| `-30%` | 文档可以拥有不匹配的可选子句总数的百分比。允许文档不匹配的子句数量被舍入到最近的整数。
组合| `2<75%` | 表达在`n<p%` 格式。如果可选子句的数量小于或等于`n`，该文档必须匹配所有可选子句。如果可选子句的数量大于`n`，然后该文档必须与`p` 可选子句的百分比。
多种组合| `3<-1 5<50%` | 一个以上的组合被空间隔开。每个条件都适用于可选子句的数量，该子句大于`<` 符号。在此示例中，如果有三个或更少的可选子句，则该文档必须匹配所有条款。如果有四个或五个可选子句，则该文档必须匹配除其中一个。如果有6个或更多可选子句，则该文档必须匹配50％。

让`n` 作为文档必须匹配的可选子句的数量。什么时候`n` 计算为一个百分比，如果`n` 小于1，然后使用1。如果`n` 大于可选子句的数量，使用可选子句的数量。
{: .note}


## 在布尔查询中使用参数

A[布尔查询]({{site.url}}{{site.baseurl}}/) 在`should` 条款和所需子句`must` 条款。可选地，它可以包含一个`filter` 子句以过滤结果。

考虑包含以下五个文档的示例索引：

```json
PUT testindex/_doc/1
{
  "text": "one OpenSearch"
}
```
{% include copy-curl.html %}

```json
PUT testindex/_doc/2
{
  "text": "one two OpenSearch"
}
```
{% include copy-curl.html %}

```json
PUT testindex/_doc/3
{
  "text": "one two three OpenSearch"
}
```
{% include copy-curl.html %}

```json
PUT testindex/_doc/4
{
  "text": "one two three four OpenSearch"
}
```
{% include copy-curl.html %}

```json
PUT testindex/_doc/5
{
  "text": "OpenSearch"
}
```
{% include copy-curl.html %}

以下查询包含四个可选子句：

```json
GET testindex/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "text": "OpenSearch"
          }
        }
      ], 
      "should": [
        {
          "match": {
            "text": "one"
          }
        },
        {
          "match": {
            "text": "two"
          }
        },
        {
          "match": {
            "text": "three"
          }
        },
        {
          "match": {
            "text": "four"
          }
        }
      ],
      "minimum_should_match": "80%"
    }
  }
}
```
{% include copy-curl.html %}

因为`minimum_should_match` 指定为`80%`，匹配的可选子句的数量计算为4＆middot;0.8 = 3.2，然后舍入至3。因此，结果包含至少匹配三个子句的文档：

```json
{
  "took": 40,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 2.494999,
    "hits": [
      {
        "_index": "testindex",
        "_id": "4",
        "_score": 2.494999,
        "_source": {
          "text": "one two three four OpenSearch"
        }
      },
      {
        "_index": "testindex",
        "_id": "3",
        "_score": 1.5744598,
        "_source": {
          "text": "one two three OpenSearch"
        }
      }
    ]
  }
}
```

现在指定`minimum_should_match` 作为`-20%`：

```json
GET testindex/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "text": "OpenSearch"
          }
        }
      ], 
      "should": [
        {
          "match": {
            "text": "one"
          }
        },
        {
          "match": {
            "text": "two"
          }
        },
        {
          "match": {
            "text": "three"
          }
        },
        {
          "match": {
            "text": "four"
          }
        }
      ],
      "minimum_should_match": "-20%"
    }
  }
}
```
{% include copy-curl.html %}

非-匹配文档可以拥有的可选条款的计算为4＆middot;0.2 = 0.8，并舍入至0。因此，结果仅包含一个与所有可选子句匹配的文档：

```json
{
  "took": 41,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 2.494999,
    "hits": [
      {
        "_index": "testindex",
        "_id": "4",
        "_score": 2.494999,
        "_source": {
          "text": "one two three four OpenSearch"
        }
      }
    ]
  }
}
```

请注意，指定正百分比（`80%`）和负百分比（`-20%`）没有导致文档必须匹配的相同数量的可选子句，因为在两种情况下，结果都已舍入。如果可选子句的数量为5，则`80%` 和`-20%` 会产生文档必须匹配的相同数量的可选子句（4）。

### 默认`minimum_should_match` 价值

如果查询包含一个`must` 或者`filter` 条款，默认`minimum_should_match` 值为0。例如，以下查询搜索匹配的文档`OpenSearch` 和0可选`should` 条款：

```json
GET testindex/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "text": "OpenSearch"
          }
        }
      ], 
      "should": [
        {
          "match": {
            "text": "one"
          }
        },
        {
          "match": {
            "text": "two"
          }
        },
        {
          "match": {
            "text": "three"
          }
        },
        {
          "match": {
            "text": "four"
          }
        }
      ]
    }
  }
}
```
{% include copy-curl.html %}

此查询返回索引中的所有五个文档：

```json
{
  "took": 34,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 5,
      "relation": "eq"
    },
    "max_score": 2.494999,
    "hits": [
      {
        "_index": "testindex",
        "_id": "4",
        "_score": 2.494999,
        "_source": {
          "text": "one two three four OpenSearch"
        }
      },
      {
        "_index": "testindex",
        "_id": "3",
        "_score": 1.5744598,
        "_source": {
          "text": "one two three OpenSearch"
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.91368985,
        "_source": {
          "text": "one two OpenSearch"
        }
      },
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.4338556,
        "_source": {
          "text": "one OpenSearch"
        }
      },
      {
        "_index": "testindex",
        "_id": "5",
        "_score": 0.11964063,
        "_source": {
          "text": "OpenSearch"
        }
      }
    ]
  }
}
```

但是，如果您省略`must` 条款，然后查询搜索匹配一个可选的文档`should` 条款：

```json
GET testindex/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "text": "one"
          }
        },
        {
          "match": {
            "text": "two"
          }
        },
        {
          "match": {
            "text": "three"
          }
        },
        {
          "match": {
            "text": "four"
          }
        }
      ]
    }
  }
}
```
{% include copy-curl.html %}

结果仅包含四个匹配至少一个可选子句的文档：

```json
{
  "took": 19,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4,
      "relation": "eq"
    },
    "max_score": 2.426633,
    "hits": [
      {
        "_index": "testindex",
        "_id": "4",
        "_score": 2.426633,
        "_source": {
          "text": "one two three four OpenSearch"
        }
      },
      {
        "_index": "testindex",
        "_id": "3",
        "_score": 1.4978898,
        "_source": {
          "text": "one two three OpenSearch"
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.8266785,
        "_source": {
          "text": "one two OpenSearch"
        }
      },
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.3331056,
        "_source": {
          "text": "one OpenSearch"
        }
      }
    ]
  }
}
```

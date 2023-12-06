---
layout: default
title: Percolator
nav_order: 65
has_children: false
grand_parent: 支持的字段类型
redirect_from:
  - /opensearch/supported-field-types/percolator/
  - /field-types/percolator/
---

# Percolator字段类型

Percolator字段类型指定将此字段视为查询。任何JSON对象字段都可以将其标记为渗滤器字段。通常，文档是索引的，并且搜索与它们相反。当您使用Percolator字段时，您可以存储搜索，然后将Percaly查询与该搜索相匹配。

## 例子

客户正在寻找价格为400美元或以下的桌子，并希望为此搜索创建警报。

创建一个将percolator字段类型分配到查询字段的映射：

```json
PUT testindex1
{
  "mappings": {
    "properties": {
      "search": {
        "properties": {
          "query": { 
            "type": "percolator" 
          }
        }
      },
      "price": { 
        "type": "float" 
      },
      "item": { 
        "type": "text" 
      }
    }
  }
}
```
{% include copy-curl.html %}

索引查询：

```json
PUT testindex1/_doc/1
{
  "search": {
    "query": {
      "bool": {
        "filter": [
          { 
            "match": { 
              "item": { 
                "query": "table" 
              }
            }
          },
          { 
            "range": { 
              "price": { 
                "lte": 400.00 
              } 
            } 
          }
        ]
      }
    }
  }
}
```
{% include copy-curl.html %}

查询中引用的字段必须已经存在于映射中。
{:.note}

运行渗透查询以搜索匹配文档：

```json
GET testindex1/_search
{
  "query" : {
    "bool" : {
      "filter" : 
        {
          "percolate" : {
            "field" : "search.query",
            "document" : {
              "item" : "Mahogany table",
              "price": 399.99
            }
          }
        }
    }
  }
}
```
{% include copy-curl.html %}

响应包含最初的索引查询：

```json
{
  "took" : 30,
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
    "max_score" : 0.0,
    "hits" : [
      {
        "_index" : "testindex1",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.0,
        "_source" : {
          "search" : {
            "query" : {
              "bool" : {
                "filter" : [
                  {
                    "match" : {
                      "item" : {
                        "query" : "table"
                      }
                    }
                  },
                  {
                    "range" : {
                      "price" : {
                        "lte" : 400.0
                      }
                    }
                  }
                ]
              }
            }
          }
        },
        "fields" : {
          "_percolator_document_slot" : [
            0
          ]
        }
      }
    ]
  }
}
```

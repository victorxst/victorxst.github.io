---
layout: default
title: 排名评估
nav_order: 60
---

# 排名评估
**引入1.0**
{: .label .label-purple }

这[秩]({{site.url}}{{site.baseurl}}/opensearch/supported-field-types/rank/) 评估端点允许您评估排名搜索结果的质量。

## 路径和HTTP方法

```
GET <index_name>/_rank_eval 
POST <index_name>/_rank_eval
```

## 查询参数

查询参数是可选的。

范围| 数据类型| 描述
:--- | :---  | :---
ignore_unavailable| 布尔| 默认为`false`。设置为`false` 如果索引关闭或丢失，响应主体将返回错误。
允许_no_indices| 布尔| 默认为`true`。设置为`false` 如果通配符表达式指向封闭或丢失的索引，则响应主体将返回错误。
Expand_WildCard| 细绳| 扩大索引的通配符表达式`open`，`closed`，`hidden`，`none`， 或者`all`。
搜索类型| 细绳| 将搜索类型设置为任一`query_then_fetch` 或者`dfs_query_then_fetch`。

## 请求字段

请求主体必须至少包含一个参数。

字段类型| 描述
:--- | :---  
ID| 文档或模板ID。
要求| 在“请求”字段部分中设置多个搜索请求。
评分| 文件相关得分。
k| 每个查询返回的文档数量。默认设置为10。
seacitant_rating_threshold| 文档被认为相关的阈值。默认设置为1。
归一化| 设置为折扣累积收益将计算`true`。
最大值| 使用预期的相互等级度量时，设置最大相关得分。
ignore_unlabeled| 默认为`false`。设置为`true`。
template_id| 模板ID。
参数| 模板中使用的参数。

#### 示例请求

````json
GET shakespeare/_rank_eval
{
  "requests": [
    {
      "id": "books_query",                        
      "request": {                                              
          "query": { "match": { "text": "thou" } }
      },
      "ratings": [                                              
        { "_index": "shakespeare", "_id": "80", "rating": 0 },
        { "_index": "shakespeare", "_id": "115", "rating": 1 },
        { "_index": "shakespeare", "_id": "117", "rating": 2 }
      ]
    },
    {
      "id": "words_query",
      "request": {
        "query": { "match": { "text": "art" } }
      },
      "ratings": [
        { "_index": "shakespeare", "_id": "115", "rating": 2 }
      ]
    }
  ]
}
````
{% include copy-curl.html %}

#### 示例响应

````json
{
  "rank_eval": {
    "metric_score": 0.7,
      "details": {
      "query_1": {                           
        "metric_score": 0.9,                      
        "unrated_docs": [                         
          {
            "_index": "shakespeare",
            "_id": "1234567"
          }, ...
        ],
        "hits": [
          {
            "hit": {                              
              "_index": "shakespeare",
              "_type": "page",
              "_id": "1234567",
              "_score": 5.123456789
            },
            "rating": 1
          }, ...
        ],
        "metric_details": {                       
          "precision": {
            "relevant_docs_retrieved": 3,
            "docs_retrieved": 6
          }
        }
      },
      "query_2": { [... ] }
    },
    "failures": { [... ] }
  }
}
````

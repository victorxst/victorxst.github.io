---
layout: default
title: 过滤器查询
nav_order: 10
has_children: false
parent: 搜索处理器
grand_parent: 搜索管道
---

# 过滤器查询处理器

这`filter_query` 搜索请求处理器拦截搜索请求，并将其他查询应用于请求，从而过滤结果。当您不想在应用程序中重写现有查询，而需要对结果进行其他过滤时，这很有用。

## 请求字段

下表列出了所有可用的请求字段。

场地| 数据类型| 描述
:--- | :--- | :---
`query` | 目的| 查询域中的查询-特定语言（DSL）。有关OpenSearch查询类型的列表，请参见[查询DSL]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/)。必需的。
`tag` | 细绳| 处理器的标识符。选修的。
`description` | 细绳| 处理器的描述。选修的。
`ignore_failure` | 布尔| 如果`true`，OpenSearch[忽略任何故障]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/creating-search-pipeline/#ignoring-processor-failures) 该处理器并继续在搜索管道中运行其余的处理器。选修的。默认为`false`。

## 例子

以下示例证明了使用搜索管道与`filter_query` 处理器。

### 设置

创建一个名称的索引`my_index` 和索引两个文件，一个公共和一家私人：

```json
POST /my_index/_doc/1
{
  "message": "This is a public message", 
  "visibility":"public"
}
```
{% include copy-curl.html %}

```json
POST /my_index/_doc/2
{
  "message": "This is a private message", 
  "visibility": "private"
}
```
{% include copy-curl.html %}

### 创建搜索管道

以下请求创建了一个称为搜索管道`my_pipeline` 与`filter_query` 请求使用术语查询仅返回公共消息的处理器：

```json
PUT /_search/pipeline/my_pipeline 
{
  "request_processors": [
    {
      "filter_query" : {
        "tag" : "tag1",
        "description" : "This processor is going to restrict to publicly visible documents",
        "query" : {
          "term": {
            "visibility": "public"
          }
        }
      }
    }
  ]
}
```
{% include copy-curl.html %}

### 使用搜索管道

搜索文档`my_index` 没有搜索管道：

```json
GET /my_index/_search
```
{% include copy-curl.html %}

响应包含两个文档：

<details open markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}
```json
{
  "took" : 47,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "my_index",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "message" : "This is a public message",
          "visibility" : "public"
        }
      },
      {
        "_index" : "my_index",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "message" : "This is a private message",
          "visibility" : "private"
        }
      }
    ]
  }
}
```
</details>

要使用管道搜索，请在`search_pipeline` 查询参数：

```json
GET /my_index/_search?search_pipeline=my_pipeline
```
{% include copy-curl.html %}

响应仅包含文档`public` 能见度：

<details open markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}
```json
{
  "took" : 19,
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
        "_index" : "my_index",
        "_id" : "1",
        "_score" : 0.0,
        "_source" : {
          "message" : "This is a public message",
          "visibility" : "public"
        }
      }
    ]
  }
}
```
</详细信息


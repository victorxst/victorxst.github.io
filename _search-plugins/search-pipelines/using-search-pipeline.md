---
layout: default
title: 使用搜索管道
nav_order: 20
has_children: false
parent: 搜索管道
grand_parent: 搜索
---

# 使用搜索管道

您可以通过以下方式使用搜索管道：

- [指定现有管道](#specifying-an-existing-search-pipeline-for-a-request) 出于请求。
- [使用临时管道](#using-a-temporary-search-pipeline-for-a-request) 出于请求。
- 设置[默认管道](#default-search-pipeline) 对于索引中的所有请求。

## 为请求指定现有的搜索管道

您先请[创建搜索管道]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/creating-search-pipeline/)，您可以通过在“查询”中指定管道名称，将管道与查询使用`search_pipeline` 查询参数：

```json
GET /my_index/_search?search_pipeline=my_pipeline
```
{％包含副本-curl.html％}

对于使用搜索管道的完整示例`filter_query` 处理器，请参阅[`filter_query` 处理器示例]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/filter-query-processor#example)。

## 使用临时搜索管道作为请求

作为创建搜索管道的替代方法，您可以定义临时搜索管道，仅用于当前查询：

```json
POST /my-index/_search
{
  "query" : {
    "match" : {
      "text_field" : "some search text"
    }
  },
  "search_pipeline" : {
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
    ],
    "response_processors": [
      {
        "rename_field": {
          "field": "message",
          "target_field": "notification"
        }
      }
    ]
  }
}
```
{％包含副本-curl.html％}

使用此语法，管道不会持续存在，仅用于指定其指定的查询。

## 默认搜索管道

为了方便起见，您可以为索引设置默认的搜索管道。索引有默认管道后，您无需指定`search_pipeline` 在每个搜索请求中查询参数。

### 为索引设置默认搜索管道

要设置索引的默认搜索管道，请指定`index.search.default_pipeline` 在索引的设置中：

```json
PUT /my_index/_settings 
{
  "index.search.default_pipeline" : "my_pipeline"
}
```
{％包含副本-curl.html％}

设置默认管道后`my_index`，您可以尝试对所有文档进行相同的搜索：

```json
GET /my_index/_search
```
{％包含副本-curl.html％}

响应仅包含公共文件，表明该管道默认情况下应用了：

<详细信息打开降价="block">
  <summary>
    回复
  </summary>
  {： 。文本-三角洲}

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
</delect>

### 禁用请求的默认管道

如果要在不应用默认管道的情况下运行搜索请求，则可以设置`search_pipeline` 查询参数为`_none`：

```json
GET /my_index/_search?search_pipeline=_none
```
{％包含副本-curl.html％}

### 删除默认管道

要从索引中删除默认管道，请将其设置为`null` 或者`_none`：

```json
PUT /my_index/_settings 
{
  "index.search.default_pipeline" : null
}
```
{％包含副本-curl.html％}

```json
PUT /my_index/_settings 
{
  "index.search.default_pipeline" : "_none"
}
```
{％包含副本-curl.html％}


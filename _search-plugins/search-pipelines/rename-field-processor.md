---
layout: default
title: 重命名字段
nav_order: 20
has_children: false
parent: 搜索处理器
grand_parent: 搜索管道
---

# 重命名现场处理器

这`rename_field` 搜索响应处理器拦截搜索响应并重命名指定字段。当您的索引和应用程序对同一字段使用不同的名称时，这很有用。例如，如果您将索引中的字段重命名为`rename_field` 处理器可以在将响应发送到您的应用程序之前将新名称更改为旧名称。

## 请求字段

下表列出了所有可用的请求字段。

场地| 数据类型| 描述
：--- | ：--- | ：---
`field` | 细绳| 重命名的字段。必需的。
`target_field` | 细绳| 新的字段名称。必需的。
`tag` | 细绳| 处理器的标识符。
`description` | 细绳| 处理器的描述。
`ignore_failure` | 布尔| 如果`true`，OpenSearch[忽略任何故障]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/creating-search-pipeline/#ignoring-processor-failures) 该处理器并继续在搜索管道中运行其余的处理器。选修的。默认为`false`。

## 例子

以下示例证明了使用搜索管道与`rename_field` 处理器。

### 设置

创建一个名称的索引`my_index` 并将文档索引`message`：

```json
POST /my_index/_doc/1
{
  "message": "This is a public message", 
  "visibility":"public"
}
```
{％包含副本-curl.html％}

### 创建搜索管道

以下请求会创建一个使用`rename_field` 重命名字段的响应处理器`message` 到`notification`：

```json
PUT /_search/pipeline/my_pipeline
{
  "response_processors": [
    {
      "rename_field": {
        "field": "message",
        "target_field": "notification"
      }
    }
  ]
}
```
{％包含副本-curl.html％}

### 使用搜索管道

搜索文档`my_index` 没有搜索管道：

```json
GET /my_index/_search
```
{％包含副本-curl.html％}

响应包含字段`message`：

<详细信息打开降价="block">
  <summary>
    回复
  </summary>
  {： 。文本-三角洲}
```json
{
  "took" : 1,
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
      }
    ]
  }
}
```
</delect>

要使用管道搜索，请在`search_pipeline` 查询参数：

```json
GET /my_index/_search?search_pipeline=my_pipeline
```
{％包含副本-curl.html％}

这`message` 现场已重命名为`notification`：

<详细信息打开降价="block">
  <summary>
    回复
  </summary>
  {： 。文本-三角洲}
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
          "visibility" : "public",
          "notification" : "This is a public message"
        }
      }
    ]
  }
}
```
</delect>

您也可以使用`fields` 在文档中搜索特定字段的选项：

```json
POST /my_index/_search?pretty&search_pipeline=my_pipeline
{
    "fields":["visibility", "message"]
}
``` 
{％包含副本-curl.html％}

在响应中，字段`message` 已重命名为`notification`：

<详细信息打开降价="block">
  <summary>
    回复
  </summary>
  {： 。文本-三角洲}
```json
{
  "took" : 4,
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
          "visibility" : "public",
          "notification" : "This is a public message"
        },
        "fields" : {
          "visibility" : [
            "public"
          ],
          "notification" : [
            "This is a public message"
          ]
        }
      }
    ]
  }
}

```
</详细信息


---
layout: default
title: 创建搜索管道
nav_order: 10
has_children: false
parent: 搜索管道
grand_parent: 搜索
---

# 创建搜索管道

搜索管道存储在群集状态。要创建搜索管道，您必须在OpenSearch集群中配置有序的处理器列表。您可以在管道中拥有多个相同类型的处理器。每个处理器都有一个`tag` 将其与其他区分开的标识符。在调试错误消息时，标记特定处理器可能会有所帮助，尤其是如果添加同一类型的多个处理器时。

#### 示例请求

以下请求会创建一个使用`filter_query` 请求处理器使用术语查询仅返回公共消息和重命名字段的响应处理器`message` 到`notification`：

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
```
{% include copy-curl.html %}

## 忽略处理器故障

默认情况下，如果其处理器之一失败，则搜索管道停止。如果您希望管道在处理器失败时继续运行，可以设置`ignore_failure` 该处理器的参数`true` 创建管道时：

```json
"filter_query" : {
  "tag" : "tag1",
  "description" : "This processor is going to restrict to publicly visible documents",
  "ignore_failure": true,
  "query" : {
    "term": {
      "visibility": "public"
    }
  }
}
```

如果处理器失败，OpenSearch日志故障并继续在搜索管道中运行所有剩余的处理器。要检查是否有任何故障，您可以使用[搜索管道指标]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/search-pipeline-metrics/)。

## 更新搜索管道

要动态更新搜索管道，请使用搜索管道API替换搜索管道。

#### 示例请求

以下示例请求upserts`my_pipeline` 通过添加一个`filter_query` 请求处理器和`rename_field` 响应处理器：

```json
PUT /_search/pipeline/my_pipeline
{
  "request_processors": [
    {
      "filter_query": {
        "tag": "tag1",
        "description": "This processor returns only publicly visible documents",
        "query": {
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
```
{% include copy-curl.html %}

## 搜索管道版本

创建管道时，您可以在`version` 范围：

```json
PUT _search/pipeline/my_pipeline
{
  "version": 1234,
  "request_processors": [
    {
      "script": {
        "source": """
           if (ctx._source['size'] > 100) {
             ctx._source['explain'] = false;
           }
         """
      }
    }
  ]
}
```
{% include copy-curl.html %}

该版本在随后的所有响应中提供`get pipeline` 要求：

```json
GET _search/pipeline/my_pipeline
```

响应包含管道版本：

<details open markdown="block">
  <summary>
    回复
  </summary>
  {： 。文本-三角洲}

```json
{
  "my_pipeline": {
    "version": 1234,
    "request_processors": [
      {
        "script": {
          "source": """
           if (ctx._source['size'] > 100) {
             ctx._source['explain'] = false;
           }
         """
        }
      }
    ]
  }
}
```
</details>


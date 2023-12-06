---
layout: default
title: 搜索管道
nav_order: 100
has_children: true
has_toc: false
---

# 搜索管道

您可以使用_search Pipelines _构建新的或重用现有结果的Reranker，查询重写器以及其他在查询或结果上运行的组件。搜索管道使您更容易在OpenSearch中处理搜索查询和搜索结果。将您的某些应用程序功能转移到OpenSearch搜索管道中会降低应用程序的整体复杂性。作为搜索管道的一部分，您指定执行模块化任务的处理器列表。然后，您可以轻松添加或重新排序这些处理器，以自定义应用程序的搜索结果。

## 术语

以下是搜索管道术语的列表：

*[_搜索请求处理器_]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/search-processors#search-request-processors)：一个截获搜索请求的组件（查询和元数据在请求中传递），对搜索请求或搜索请求执行操作，然后返回搜索请求。
*[_搜索响应处理器_]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/search-processors#search-response-processors)：拦截搜索响应和搜索请求的组件（在请求中传递的查询，结果和元数据），对搜索响应进行操作，并返回搜索响应。
*[_搜索阶段结果处理器_]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/search-processors#search-phase-results-processors)：在协调节点级别上搜索阶段之间运行的组件。搜索阶段结果处理器拦截了从一个搜索阶段检索到的结果，并将其转换为下一个搜索阶段。
*[_处理器_]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/search-processors/)：搜索请求处理器或搜索响应处理器。
* _ Search Pipeline_：集成到OpenSearch中的处理器的有序列表。管道截止查询，在查询上执行处理，将其发送到OpenSearch，拦截结果，对结果进行处理，并将其返回到调用应用程序中，如下图所示。

![搜索处理器图]({{site.url}}{{site.baseurl}}/images/search-pipelines.png)

管道的请求和响应处理都在协调器节点上执行，因此没有碎片-水平处理。
{:.note}

## 处理器

要了解有关可用搜索处理器的更多信息，请参阅[搜索处理器]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/search-processors/)。


## 例子

要创建搜索管道，请将请求发送到搜索管道端点，指定有序的处理器列表，该列表将依次应用：

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

有关创建和更新搜索管道的更多信息，请参见[创建搜索管道]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/creating-search-pipeline/)。

要使用查询的管道，请在此处指定管道名称`search_pipeline` 查询参数：

```json
GET /my_index/_search?search_pipeline=my_pipeline
```
{% include copy-curl.html %}

另外，您可以使用请求的临时管道或为索引设置默认管道。要了解更多，请参阅[使用搜索管道]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/using-search-pipeline/)。

要了解检索现有搜索管道的详细信息，请参阅[检索搜索管道]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/retrieving-search-pipeline/)。


## 搜索管道指标

有关检索搜索管道统计信息的信息，请参见[搜索管道指标]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/search-pipeline-metrics/)


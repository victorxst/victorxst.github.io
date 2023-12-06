---
layout: default
title: 搜索处理器
nav_order: 40
has_children: true
parent: 搜索管道
grand_parent: 搜索
---

# 搜索处理器

搜索处理器可以是以下类型：

- [搜索请求处理器](#search-request-processors)
- [搜索响应处理器](#search-response-processors)
- [搜索阶段结果处理器](#search-phase-results-processors)

## 搜索请求处理器

搜索请求处理器拦截了搜索请求（查询和请求中传递的元数据），对搜索请求或搜索请求执行操作，并将搜索请求提交索引。

下表列出了所有受支持的搜索请求处理器。

处理器| 描述| 最早的可用版本
:--- | :--- | :---
[`filter_query`]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/filter-query-processor/) | 添加了用于过滤请求的过滤查询。| 2.8
[`neural_query_enricher`]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/neural-query-enricher/) | 在索引或字段级别设置用于神经搜索的默认模型。| 2.11
[`script`]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/script-processor/) | 添加了一个在新索引的文档上运行的脚本。| 2.8

## 搜索响应处理器

搜索响应处理器拦截了搜索响应和搜索请求（在请求中传递的查询，结果和元数据），对搜索响应或搜索响应执行操作，并返回搜索响应。

下表列出了所有支持的搜索响应处理器。

处理器| 描述| 最早的可用版本
:--- | :--- | :---
[`personalize_search_ranking`]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/personalize-search-ranking/) | 用途[亚马逊个性化](https://aws.amazon.com/personalize/) 要重读搜索结果（需要设置Amazon个性化服务）。| 2.9
[`rename_field`]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/rename-field-processor/)| 重命名现有字段。| 2.8

## 搜索阶段结果处理器

搜索阶段结果处理器在协调节点级别的搜索阶段之间运行。它拦截了从一个搜索阶段检索到的结果，并将其转换为下一个搜索阶段。

下表列出了所有受支持的搜索请求处理器。

处理器| 描述| 最早的可用版本
:--- | :--- | :---
[`normalization-processor`]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/normalization-processor/) | 拦截查询阶段结果并归一化并结合文档分数，然后再将文档传递给获取阶段。| 2.10

## 查看可用的处理器类型

您可以使用节点搜索管道API查看可用处理器类型：

```json
GET /_nodes/search_pipelines
```
{% include copy-curl.html %}

响应包含`search_pipelines` 列出可用请求和响应处理器的对象：

<details open markdown="block">
  <summary>
    回复
  </summary>
  {: .text-delta}

```json
{
  "_nodes" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "cluster_name" : "runTask",
  "nodes" : {
    "36FHvCwHT6Srbm2ZniEPhA" : {
      "name" : "runTask-0",
      "transport_address" : "127.0.0.1:9300",
      "host" : "127.0.0.1",
      "ip" : "127.0.0.1",
      "version" : "3.0.0",
      "build_type" : "tar",
      "build_hash" : "unknown",
      "roles" : [
        "cluster_manager",
        "data",
        "ingest",
        "remote_cluster_client"
      ],
      "attributes" : {
        "testattr" : "test",
        "shard_indexing_pressure_enabled" : "true"
      },
      "search_pipelines" : {
        "request_processors" : [
          {
            "type" : "filter_query"
          },
          {
            "type" : "script"
          }
        ],
        "response_processors" : [
          {
            "type" : "rename_field"
          }
        ]
      }
    }
  }
}
```
</details>

除了OpenSearch提供的处理器外，插件还可以提供其他处理器。
{: .note}


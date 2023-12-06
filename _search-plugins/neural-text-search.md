---
layout: default
title: 文字搜索
nav_order: 10
has_children: false
parent: 神经搜索
---

# 神经文本搜索

使用文本搜索文本数据。在神经搜索中，文本搜索是通过文本嵌入模型来促进的。文本搜索创建一个密集的向量（浮子列表）并将数据摄入k-NN索引。

**先决条件**<br>
在使用文本搜索之前，必须设置文本嵌入模型。有关更多信息，请参阅[在OpenSearch中使用ML模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/) 和[连接到远程型号]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/index/)。
{:.note}

## 使用文本搜索

要使用文本搜索，请执行以下步骤：

1. [创建摄入管道](#step-1-create-an-ingest-pipeline)。
1. [创建摄入索引](#step-2-create-an-index-for-ingestion)。
1. [摄取索引的文档](#step-3-ingest-documents-into-the-index)。
1. [使用神经搜索搜索索引](#step-4-search-the-index-using-neural-search)。

## 步骤1：创建一个摄入管道

要生成向量嵌入，您需要创建一个[摄入管道]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/index/) 其中包含一个[`text_embedding` 处理器]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/processors/text-embedding/)，这将将文档字段中的文本转换为向量嵌入。处理器的`field_map` 确定从中生成向量嵌入的输入字段以及用于存储嵌入的输出字段。

以下示例请求创建了一个摄入的管道，其中文本来自`passage_text` 将转换为文本嵌入，嵌入将存储在`passage_embedding`：

```json
PUT /_ingest/pipeline/nlp-ingest-pipeline
{
  "description": "A text embedding pipeline",
  "processors": [
    {
      "text_embedding": {
        "model_id": "bQ1J8ooBpBj3wT4HVUsb",
        "field_map": {
          "passage_text": "passage_embedding"
        }
      }
    }
  ]
}
```
{% include copy-curl.html %}

## 步骤2：创建摄入索引

为了使用管道中定义的文本嵌入处理器，创建k-NN索引，将上一步中创建的管道添加为默认管道。确保在`field_map` 被映射为正确的类型。继续以示例为例`passage_embedding` 字段必须映射为k-NN向量具有与模型维度相匹配的维数。同样，`passage_text` 字段应映射为`text`。

以下示例请求创建k-使用默认摄入管道设置的NN索引：

```json
PUT /my-nlp-index
{
  "settings": {
    "index.knn": true,
    "default_pipeline": "nlp-ingest-pipeline"
  },
  "mappings": {
    "properties": {
      "id": {
        "type": "text"
      },
      "passage_embedding": {
        "type": "knn_vector",
        "dimension": 768,
        "method": {
          "engine": "lucene",
          "space_type": "l2",
          "name": "hnsw",
          "parameters": {}
        }
      },
      "passage_text": {
        "type": "text"
      }
    }
  }
}
```
{% include copy-curl.html %}

有关创建K的更多信息-NN索引及其支持的方法，请参阅[k-NN索引]({{site.url}}{{site.baseurl}}/search-plugins/knn/knn-index/)。

## 步骤3：将文档摄入索引

要将文档摄取到上一步中创建的索引中，请发送以下请求：

```json
PUT /my-nlp-index/_doc/1
{
  "passage_text": "Hello world",
  "id": "s1"
}
```
{% include copy-curl.html %}

```json
PUT /my-nlp-index/_doc/2
{
  "passage_text": "Hi planet",
  "id": "s2"
}
```
{% include copy-curl.html %}

在将文档摄入索引之前，摄入管道运行`text_embedding` 文档上的处理器，生成文本嵌入`passage_text` 场地。索引文档包括`passage_text` 字段，其中包含原始文本，`passage_embedding` 字段，其中包含向量嵌入。

## 步骤4：使用神经搜索搜索索引

要在您的索引上执行向量搜索，请使用`neural` 查询子句在[k-NN插件API]({{site.url}}{{site.baseurl}}/search-plugins/knn/api/#search-model) 或者[查询DSL]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/index/) 查询。您可以使用一个[k-NN搜索过滤器]({{site.url}}{{site.baseurl}}/search-plugins/knn/filter-search-knn/)。

以下示例请求使用布尔查询来组合过滤器子句和两个查询条款---神经查询和一个`match` 询问。这`script_score` 查询将自定义权重分配给查询条款：

```json
GET /my-nlp-index/_search
{
  "_source": {
    "excludes": [
      "passage_embedding"
    ]
  },
  "query": {
    "bool": {
      "filter": {
         "wildcard":  { "id": "*1" }
      },
      "should": [
        {
          "script_score": {
            "query": {
              "neural": {
                "passage_embedding": {
                  "query_text": "Hi world",
                  "model_id": "bQ1J8ooBpBj3wT4HVUsb",
                  "k": 100
                }
              }
            },
            "script": {
              "source": "_score * 1.5"
            }
          }
        },
        {
          "script_score": {
            "query": {
              "match": {
                "passage_text": "Hi world"
              }
            },
            "script": {
              "source": "_score * 1.7"
            }
          }
        }
      ]
    }
  }
}
```
{% include copy-curl.html %}

响应包含匹配文档：

```json
{
  "took" : 36,
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
    "max_score" : 1.2251667,
    "hits" : [
      {
        "_index" : "my-nlp-index",
        "_id" : "1",
        "_score" : 1.2251667,
        "_source" : {
          "passage_text" : "Hello world",
          "id" : "s1"
        }
      }
    ]
  }
}
```

## 在索引或字段上设置默认模型

A[`neural`]({{site.url}}{{site.baseurl}}/query-dsl/specialized/neural/) 查询需要一个用于生成向量嵌入的模型ID。为了消除每个神经查询请求的传递模型ID，您可以在k上设置默认模型-NN索引或字段。

首先，创建一个[搜索管道]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/index/) 与[`neural_query_enricher`]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/neural-query-enricher/) 请求处理器。要为索引设置默认模型，请在`default_model_id` 范围。要为特定字段设置默认模型，请在“`neural_field_default_id` 地图。如果您提供两者`default_model_id` 和`neural_field_default_id`，`neural_field_default_id` 优先：

```json
PUT /_search/pipeline/default_model_pipeline 
{
  "request_processors": [
    {
      "neural_query_enricher" : {
        "default_model_id": "bQ1J8ooBpBj3wT4HVUsb",
        "neural_field_default_id": {
           "my_field_1": "uZj0qYoBMtvQlfhaYeud",
           "my_field_2": "upj0qYoBMtvQlfhaZOuM"
        }
      }
    }
  ]
}
```
{% include copy-curl.html %}

然后为您的索引设置默认模型：

```json
PUT /my-nlp-index/_settings
{
  "index.search.default_pipeline" : "default_model_pipeline"
}
```
{% include copy-curl.html %}

现在，您可以在搜索时省略模型ID：

```json
GET /my-nlp-index/_search
{
  "_source": {
    "excludes": [
      "passage_embedding"
    ]
  },
  "query": {
    "neural": {
      "passage_embedding": {
        "query_text": "Hi world",
        "k": 100
      }
    }
  }
}
```
{% include copy-curl.html %}

响应包含两个文档：

```json
{
  "took" : 41,
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
    "max_score" : 1.22762,
    "hits" : [
      {
        "_index" : "my-nlp-index",
        "_id" : "2",
        "_score" : 1.22762,
        "_source" : {
          "passage_text" : "Hi planet",
          "id" : "s2"
        }
      },
      {
        "_index" : "my-nlp-index",
        "_id" : "1",
        "_score" : 1.2251667,
        "_source" : {
          "passage_text" : "Hello world",
          "id" : "s1"
        }
      }
    ]
  }
}
```

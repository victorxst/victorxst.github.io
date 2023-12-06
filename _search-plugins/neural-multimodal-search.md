---
layout: default
title: 多模式搜索
nav_order: 20
has_children: false
parent: 神经搜索
---

# 多模式搜索
引入2.11
{: .label .label-purple }

使用多模式搜索搜索文本和图像数据。在神经搜索中，文本搜索是通过多模式嵌入模型来促进的。

**先决条件**<br>
在使用文本搜索之前，您必须设置多模式嵌入模型。有关更多信息，请参阅[在OpenSearch中使用ML模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/) 和[连接到远程型号]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/index/)。
{:.note}

## 使用多模式搜索

要使用文本和图像嵌入的神经搜索，请按照以下步骤：

1. [创建摄入管道](#step-1-create-an-ingest-pipeline)。
1. [创建摄入索引](#step-2-create-an-index-for-ingestion)。
1. [摄取索引的文档](#step-3-ingest-documents-into-the-index)。
1. [使用神经搜索搜索索引](#step-4-search-the-index-using-neural-search)。

## 步骤1：创建一个摄入管道

要生成向量嵌入，您需要创建一个[摄入管道]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/index/) 其中包含一个[`text_image_embedding` 处理器]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/processors/text-image-embedding/)，将在文档字段中将文本或图像转换为向量嵌入。处理器的`field_map` 确定可以从中生成向量嵌入的文本和图像字段，以及用于存储嵌入的输出向量字段。

以下示例请求创建了一个摄入的管道，其中文本来自`image_description` 以及来自`image_binary` 将转换为文本嵌入，嵌入将存储在`vector_embedding`：

```json
PUT /_ingest/pipeline/nlp-ingest-pipeline
{
  "description": "A text/image embedding pipeline",
  "processors": [
    {
      "text_image_embedding": {
        "model_id": "-fYQAosBQkdnhhBsK593",
        "embedding": "vector_embedding",
        "field_map": {
          "text": "image_description",
          "image": "image_binary"
        }
      }
    }
  ]
}
```
{% include copy-curl.html %}

## 步骤2：创建摄入索引

为了使用管道中定义的文本嵌入处理器，创建k-NN索引，将上一步中创建的管道添加为默认管道。确保在`field_map` 被映射为正确的类型。继续以示例为例`vector_embedding` 字段必须映射为k-NN向量具有与模型维度相匹配的维数。同样，`image_description` 字段应映射为`text`和`image_binary` 应该映射为`binary`。

以下示例请求创建k-使用默认摄入管道设置的NN索引：

```json
PUT /my-nlp-index
{
  "settings": {
    "index.knn": true,
    "default_pipeline": "nlp-ingest-pipeline",
    "number_of_shards": 2
  },
  "mappings": {
    "properties": {
      "vector_embedding": {
        "type": "knn_vector",
        "dimension": 1024,
        "method": {
          "name": "hnsw",
          "engine": "lucene",
          "parameters": {}
        }
      },
      "image_description": {
        "type": "text"
      },
      "image_binary": {
        "type": "binary"
      }
    }
  }
}
```
{% include copy-curl.html %}

有关创建K的更多信息-NN索引及其支持的方法，请参阅[k-NN索引]({{site.url}}{{site.baseurl}}/search-plugins/knn/knn-index/)。

## 步骤3：将文档摄入索引

要将文档摄入上一步中创建的索引，请发送以下请求：

```json
PUT /nlp-index/_doc/1
{
 "image_description": "Orange table",
 "image_binary": "iVBORw0KGgoAAAANSUI..."
}
```
{% include copy-curl.html %}

在将文档摄入索引之前，摄入管道运行`text_image_embedding` 文档上的处理器，生成矢量嵌入`image_description` 和`image_binary` 字段。除了原始`image_description` 和`image_binary` 字段，索引文档包括`vector_embedding` 字段，其中包含组合的向量嵌入。

## 步骤4：使用神经搜索搜索索引

要在您的索引上执行向量搜索，请使用`neural` 查询子句在[k-NN插件API]({{site.url}}{{site.baseurl}}/search-plugins/knn/api/#search-model) 或者[查询DSL]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/index/) 查询。您可以使用一个[k-NN搜索过滤器]({{site.url}}{{site.baseurl}}/search-plugins/knn/filter-search-knn/)。您可以通过文本，图像或文本和图像进行搜索。

以下示例请求使用神经查询搜索文本和图像：

```json
GET /my-nlp-index/_search
{
  "size": 10,
  "query": {
    "neural": {
      "vector_embedding": {
        "query_text": "Orange table",
        "query_image": "iVBORw0KGgoAAAANSUI...",
        "model_id": "-fYQAosBQkdnhhBsK593",
        "k": 5
      }
    }
  }
}
```
{% include copy-curl.html %}

为了消除每个神经查询请求的传递模型ID，您可以在k上设置默认模型-NN索引或字段。要了解更多，请参阅[在索引或字段上设置默认模型]({{site.url}}{{site.baseurl}}/search-plugins/neural-text-search/##setting-a-default-model-on-an-index-or-field)。


---
layout: default
title: 语义搜索
has_children: false
nav_order: 140
---

# 语义搜索

默认情况下，OpenSearch使用[Okapi BM25](https://en.wikipedia.org/wiki/Okapi_BM25) 算法。BM25是关键字-基于包含关键字但无法捕获查询术语的语义含义的查询的基于算法。语义搜索，与关键字不同-基于搜索，考虑搜索上下文中查询的含义。因此，当查询需要自然语言理解时，语义搜索的性能很好。

在本教程中，您将学习如何：

- 在OpenSearch中实现语义搜索。
- 通过结合语义和关键字搜索来提高搜索相关性来实现混合搜索。

## 术语

在启动本教程之前，了解以下条款很有帮助：

- _smantic Search_：采用神经搜索来确定用户查询在搜索上下文中的意图并提高搜索相关性。
- _ Neural Search_：促进在摄入时和搜索时的矢量搜索：
  - 在摄入时，神经搜索使用语言模型来从文档中的文本字段生成矢量嵌入。然后将包含原始文本字段和矢量嵌入字段的文档索引在k中-NN索引，如下图所示。

  ![摄入时间图时的神经搜索]({{site.url}}{{site.baseurl}}/images/neural-search-ingestion.png)
  - 在搜索时间，当您使用_neural Query_时，查询文本将通过语言模型传递，并将结果向量的嵌入与文档文本矢量嵌入式进行比较，以找到最相关的结果，如下图所示。

  ![搜索时间图的神经搜索]({{site.url}}{{site.baseurl}}/images/neural-search-query.png)
- _HYBRID搜索_：结合语义和关键字搜索以提高搜索相关性。

## 语义搜索的OpenSearch组件

在本教程中，您将使用以下OpenSearch组件实现语义搜索：

- [模型组]({{site.url}}{{site.baseurl}}/ml-commons-plugin/model-access-control#model-groups)
- [OpenSearch提供的验证语言模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/pretrained-models/)
- [摄入管道]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/index/)
- [k-nn矢量]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/knn-vector/)
- [神经搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-search/)
- [搜索管道]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/index/)
- [归一化处理器]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/normalization-processor/)
- [混合查询]({{site.url}}{{site.baseurl}}/query-dsl/compound/hybrid/)

在遵循本教程时，您会发现所有这些组件的描述，因此，如果您不熟悉其中的一些内容，请不要担心。前面列表中的每个链接都将带您到相应组件的文档部分。

## 先决条件

对于此简单的设置，您将使用OpenSearch-提供机器学习（ML）模型和没有专用ML节点的群集。为确保此基本本地设置有效，请发送以下请求以更新ML-相关集群设置：

```json
PUT _cluster/settings
{
  "persistent": {
    "plugins": {
      "ml_commons": {
        "only_run_on_ml_node": "false",
        "model_access_control_enabled": "true",
        "native_memory_threshold": "99"
      }
    }
  }
}
```
{% include copy-curl.html %}

#### 先进的

对于更高级的设置，请注意以下要求：

- 要注册自定义模型，您需要指定其他`"allow_registering_model_via_url": "true"` 集群设置。
- 在生产中，最好的做法是通过专用ML节点分开工作量。在具有专用ML节点的群集上，指定`"only_run_on_ml_node": "true"` 以提高性能。

有关ML的更多信息-相关集群设置，请参阅[ML Commons集群设置]({{site.url}}{{site.baseurl}}/ml-commons-plugin/cluster-settings/)。

## 教程概述

该教程包括以下步骤：

1. [**设置ML语言模型**](#step-1-set-up-an-ml-language-model)。
    1。[选择语言模型](#step-1a-choose-a-language-model)。
    1。[注册模型组](#step-1b-register-a-model-group)。
    1。[将模型注册到模型组](#step-1c-register-the-model-to-the-model-group)。
    1。[部署模型](#step-1d-deploy-the-model)。
1. [**通过神经搜索摄取数据**](#step-2-ingest-data-with-neural-search)。
    1。[创建用于神经搜索的摄入管道](#step-2a-create-an-ingest-pipeline-for-neural-search)。
    1。[创建一个k-NN索引](#step-2b-create-a-k-nn-index)。
    1。[摄取索引的文档](#step-2c-ingest-documents-into-the-index)。
1. [**搜索数据**](#step-3-search-the-data)。
   - [使用关键字搜索搜索](#search-using-a-keyword-search)。
   - [使用神经搜索搜索](#search-using-a-neural-search)。
   - [使用混合搜索搜索](#search-using-a-hybrid-search)。

教程中的一些步骤包含可选的`Test it` 部分。您可以通过在这些部分中运行请求来确保步骤成功。

完成后，请按照[清理](#clean-up) 删除所有创建组件的部分。

## 教程

您可以使用命令行或OpenSearch仪表板关注本教程[开发工具控制台]({{site.url}}{{site.baseurl}}/dashboards/dev-tools/run-queries/)。

## 步骤1：设置ML语言模型

神经搜索需要语言模型，以便在摄入时间和查询时间从文本字段生成向量嵌入。

### 步骤1（a）：选择语言模型

对于本教程，您将使用[Distilbert](https://huggingface.co/docs/transformers/model_doc/distilbert) 网上的模型。它是OpenSearch中可用的句子句子变压器模型之一，它在基准测试中显示了一些最佳结果（有关详细信息，请参阅[这篇博客文章](https://opensearch.org/blog/semantic-science-benchmarks/)）。您需要模型的名称，版本和维度来注册。您可以在[预验证的模型表]({{site.url}}{{site.baseurl}}/ml-commons-plugin/pretrained-models/#sentence-transformers) 通过选择`config_url` 链接对应于模型的Torchscript伪像：

- 模型名称是`huggingface/sentence-transformers/msmarco-distilbert-base-tas-b`。
- 模型版本是`1.0.1`。
- 该模型的尺寸数为`768`。

#### 高级：使用不同的模型

或者，您可以选择使用其中一种[OpenSearch提供的验证语言模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/pretrained-models/) 或您自己的自定义模型。有关选择模型的信息，请参阅[进一步阅读](#further-reading)。有关如何设置自定义模型的说明，请参见[在OpenSearch中使用ML模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/)。

注意模型的维度，因为设置k时需要它-NN索引。
{： 。重要的}

### 步骤1（b）：注册模型组

对于访问控制，模型被组织成模型组（特定模型的版本的集合）。集群中的每个模型组名称必须在全球范围内唯一。注册模型组可确保模型组名称的独特性。

如果您在不首先注册模型组的情况下注册模型的第一个版本，则会自动创建一个新的模型组。有关更多信息，请参阅[模型访问控制]({{site.url}}{{site.baseurl}}/ml-commons-plugin/model-access-control/)。
{: .tip}

注册一个模型组，以访问模式设置为`public`，发送以下请求：

```json
POST /_plugins/_ml/model_groups/_register
{
  "name": "NLP_model_group",
  "description": "A model group for NLP models",
  "access_mode": "public"
}
```
{% include copy-curl.html %}

OpenSearch还会发送模型组ID：

```json
{
  "model_group_id": "Z1eQf4oB5Vm0Tdw8EIP2",
  "status": "CREATED"
}
```

您将使用此ID将所选模型注册到模型组。

<详细信息关闭的markdown ="block">
  <summary>
    测试它
  </summary>
  {: .text-delta}

通过在请求中提供其模型组ID来搜索新创建的模型组：

```json
POST /_plugins/_ml/model_groups/_search
{
  "query": {
    "match": {
      "_id": "Z1eQf4oB5Vm0Tdw8EIP2"
    }
  }
}
```
{% include copy-curl.html %}

响应包含模型组：

```json
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": ".plugins-ml-model-group",
        "_id": "Z1eQf4oB5Vm0Tdw8EIP2",
        "_version": 1,
        "_seq_no": 14,
        "_primary_term": 2,
        "_score": 1,
        "_source": {
          "created_time": 1694357262582,
          "access": "public",
          "latest_version": 0,
          "last_updated_time": 1694357262582,
          "name": "NLP_model_group",
          "description": "A model group for NLP models"
        }
      }
    ]
  }
}
```
</details>


### 步骤1（c）：将模型注册到模型组

要向模型组注册模型，请在注册请求中提供模型组ID：

```json
POST /_plugins/_ml/models/_register
{
  "name": "huggingface/sentence-transformers/msmarco-distilbert-base-tas-b",
  "version": "1.0.1",
  "model_group_id": "Z1eQf4oB5Vm0Tdw8EIP2",
  "model_format": "TORCH_SCRIPT"
}
```
{% include copy-curl.html %}

注册模型是一项异步任务。OpenSearch向此任务发送了一个任务ID：

```json
{
  "task_id": "aFeif4oB5Vm0Tdw8yoN7",
  "status": "CREATED"
}
```

OpenSearch从URL下载模型的配置文件和模型内容。由于该模型的尺寸大于10 MB，因此OpenSearch将其分成多达10 MB的块，并将这些块保存在模型索引中。您可以使用任务API检查任务的状态：

```json
GET /_plugins/_ml/tasks/aFeif4oB5Vm0Tdw8yoN7
```
{% include copy-curl.html %}

任务完成后，任务状态将是`COMPLETED` 任务API响应将包含注册模型的模型ID：

```json
{
  "model_id": "aVeif4oB5Vm0Tdw8zYO2",
  "task_type": "REGISTER_MODEL",
  "function_name": "TEXT_EMBEDDING",
  "state": "COMPLETED",
  "worker_node": [
    "4p6FVOmJRtu3wehDD74hzQ"
  ],
  "create_time": 1694358489722,
  "last_update_time": 1694358499139,
  "is_async": true
}
```

您需要模型ID才能将此模型用于以下几个步骤。

<详细信息关闭的markdown ="block">
  <summary>
    测试它
  </summary>
  {: .text-delta}

通过在请求中提供其ID来搜索新创建的模型：

```json
GET /_plugins/_ml/models/aVeif4oB5Vm0Tdw8zYO2
```
{% include copy-curl.html %}

响应包含模型：

```json
{
  "name": "huggingface/sentence-transformers/msmarco-distilbert-base-tas-b",
  "model_group_id": "Z1eQf4oB5Vm0Tdw8EIP2",
  "algorithm": "TEXT_EMBEDDING",
  "model_version": "1",
  "model_format": "TORCH_SCRIPT",
  "model_state": "REGISTERED",
  "model_content_size_in_bytes": 266352827,
  "model_content_hash_value": "acdc81b652b83121f914c5912ae27c0fca8fabf270e6f191ace6979a19830413",
  "model_config": {
    "model_type": "distilbert",
    "embedding_dimension": 768,
    "framework_type": "SENTENCE_TRANSFORMERS",
    "all_config": """{"_name_or_path":"old_models/msmarco-distilbert-base-tas-b/0_Transformer","activation":"gelu","architectures":["DistilBertModel"],"attention_dropout":0.1,"dim":768,"dropout":0.1,"hidden_dim":3072,"initializer_range":0.02,"max_position_embeddings":512,"model_type":"distilbert","n_heads":12,"n_layers":6,"pad_token_id":0,"qa_dropout":0.1,"seq_classif_dropout":0.2,"sinusoidal_pos_embds":false,"tie_weights_":true,"transformers_version":"4.7.0","vocab_size":30522}"""
  },
  "created_time": 1694482261832,
  "last_updated_time": 1694482324282,
  "last_registered_time": 1694482270216,
  "last_deployed_time": 1694482324282,
  "total_chunks": 27,
  "planning_worker_node_count": 1,
  "current_worker_node_count": 1,
  "planning_worker_nodes": [
    "4p6FVOmJRtu3wehDD74hzQ"
  ],
  "deploy_to_all_nodes": true
}
```

响应包含模型信息。您可以看到`model_state` 是`REGISTERED`。此外，该模型分为27个块，如图所示`total_chunks` 场地。
</delect>

#### 高级：注册自定义模型

要注册自定义模型，您必须在注册请求中提供模型配置。例如，以下是包含本教程中使用的模型的完整格式的寄存器请求：

```json
POST /_plugins/_ml/models/_register
{
  "name": "huggingface/sentence-transformers/msmarco-distilbert-base-tas-b",
  "version": "1.0.1",
  "model_group_id": "Z1eQf4oB5Vm0Tdw8EIP2",
  "description": "This is a port of the DistilBert TAS-B Model to sentence-transformers model: It maps sentences & paragraphs to a 768 dimensional dense vector space and is optimized for the task of semantic search.",
  "model_task_type": "TEXT_EMBEDDING",
  "model_format": "TORCH_SCRIPT",
  "model_content_size_in_bytes": 266352827,
  "model_content_hash_value": "acdc81b652b83121f914c5912ae27c0fca8fabf270e6f191ace6979a19830413",
  "model_config": {
    "model_type": "distilbert",
    "embedding_dimension": 768,
    "framework_type": "sentence_transformers",
    "all_config": """{"_name_or_path":"old_models/msmarco-distilbert-base-tas-b/0_Transformer","activation":"gelu","architectures":["DistilBertModel"],"attention_dropout":0.1,"dim":768,"dropout":0.1,"hidden_dim":3072,"initializer_range":0.02,"max_position_embeddings":512,"model_type":"distilbert","n_heads":12,"n_layers":6,"pad_token_id":0,"qa_dropout":0.1,"seq_classif_dropout":0.2,"sinusoidal_pos_embds":false,"tie_weights_":true,"transformers_version":"4.7.0","vocab_size":30522}"""
  },
  "created_time": 1676073973126
}
```

有关更多信息，请参阅[在OpenSearch中使用ML模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/)。

### 步骤1（d）：部署模型

注册模型后，将其保存在模型索引中。接下来，您需要部署模型。部署模型会创建模型实例并在内存中缓存模型。要部署模型，将其模型ID提供给`_deploy` 端点：

```json
POST /_plugins/_ml/models/aVeif4oB5Vm0Tdw8zYO2/_deploy
```
{% include copy-curl.html %}

像寄存器操作一样，部署操作是异步的，因此您将在响应中获得任务ID：

```json
{
  "task_id": "ale6f4oB5Vm0Tdw8NINO",
  "status": "CREATED"
}
```

您可以使用任务API检查任务的状态：

```json
GET /_plugins/_ml/tasks/ale6f4oB5Vm0Tdw8NINO
```
{% include copy-curl.html %}

任务完成后，任务状态将是`COMPLETED`：

```json
{
  "model_id": "aVeif4oB5Vm0Tdw8zYO2",
  "task_type": "DEPLOY_MODEL",
  "function_name": "TEXT_EMBEDDING",
  "state": "COMPLETED",
  "worker_node": [
    "4p6FVOmJRtu3wehDD74hzQ"
  ],
  "create_time": 1694360024141,
  "last_update_time": 1694360027940,
  "is_async": true
}
```

<详细信息关闭的markdown ="block">
  <summary>
    测试它
  </summary>
  {: .text-delta}

通过在请求中提供其ID来搜索部署的模型：

```json
GET /_plugins/_ml/models/aVeif4oB5Vm0Tdw8zYO2
```
{% include copy-curl.html %}

响应显示模型状态为`DEPLOYED`：

```json
{
  "name": "huggingface/sentence-transformers/msmarco-distilbert-base-tas-b",
  "model_group_id": "Z1eQf4oB5Vm0Tdw8EIP2",
  "algorithm": "TEXT_EMBEDDING",
  "model_version": "1",
  "model_format": "TORCH_SCRIPT",
  "model_state": "DEPLOYED",
  "model_content_size_in_bytes": 266352827,
  "model_content_hash_value": "acdc81b652b83121f914c5912ae27c0fca8fabf270e6f191ace6979a19830413",
  "model_config": {
    "model_type": "distilbert",
    "embedding_dimension": 768,
    "framework_type": "SENTENCE_TRANSFORMERS",
    "all_config": """{"_name_or_path":"old_models/msmarco-distilbert-base-tas-b/0_Transformer","activation":"gelu","architectures":["DistilBertModel"],"attention_dropout":0.1,"dim":768,"dropout":0.1,"hidden_dim":3072,"initializer_range":0.02,"max_position_embeddings":512,"model_type":"distilbert","n_heads":12,"n_layers":6,"pad_token_id":0,"qa_dropout":0.1,"seq_classif_dropout":0.2,"sinusoidal_pos_embds":false,"tie_weights_":true,"transformers_version":"4.7.0","vocab_size":30522}"""
  },
  "created_time": 1694482261832,
  "last_updated_time": 1694482324282,
  "last_registered_time": 1694482270216,
  "last_deployed_time": 1694482324282,
  "total_chunks": 27,
  "planning_worker_node_count": 1,
  "current_worker_node_count": 1,
  "planning_worker_nodes": [
    "4p6FVOmJRtu3wehDD74hzQ"
  ],
  "deploy_to_all_nodes": true
}
```

您还可以通过发送一个[模型配置文件API]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/profile/) 要求：

```json
GET /_plugins/_ml/profile/models
```
</details>

## 步骤2：通过神经搜索获取数据

神经搜索使用语言模型将文本转换为矢量嵌入。在摄入期间，神经搜索会在请求中为文本字段创建矢量嵌入。在搜索过程中，您可以通过应用相同的模型来生成查询文本的矢量嵌入，从而使您可以在文档上执行矢量相似性搜索。

### 步骤2（a）：创建用于神经搜索的摄入管道

现在您已经部署了一个模型，可以使用此模型来配置[神经搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-search/)。首先，您需要创建一个[摄入管道]({{site.url}}{{site.baseurl}}/api-reference/ingest-apis/index/) 其中包含一个处理器：在将文档摄入索引之前转换文档字段的任务。对于神经搜索，您将设置一个`text_embedding` 从文本创建向量嵌入的处理器。您需要`model_id` 您在上一节中设置的模型和`field_map`，指定从中获取文本的字段的名称（`text`）以及记录嵌入的字段名称（`passage_embedding`）：

```json
PUT /_ingest/pipeline/nlp-ingest-pipeline
{
  "description": "An NLP ingest pipeline",
  "processors": [
    {
      "text_embedding": {
        "model_id": "aVeif4oB5Vm0Tdw8zYO2",
        "field_map": {
          "text": "passage_embedding"
        }
      }
    }
  ]
}
```
{% include copy-curl.html %}

<详细信息关闭的markdown ="block">
  <summary>
    测试它
  </summary>
  {: .text-delta}

通过使用Ingest API搜索创建的摄入管道：

```json
GET /_ingest/pipeline
```
{% include copy-curl.html %}

响应包含摄入管道：

```json
{
  "nlp-ingest-pipeline": {
    "description": "An NLP ingest pipeline",
    "processors": [
      {
        "text_embedding": {
          "model_id": "aVeif4oB5Vm0Tdw8zYO2",
          "field_map": {
            "text": "passage_embedding"
          }
        }
      }
    ]
  }
}
```
</details>

### 步骤2（b）：创建K-NN索引

现在您将创建一个K-NN索引，带有名称的字段`text`，其中包含图像描述，一个[`knn_vector`]({{site.url}}{{site.baseurl}}/field-types/supported-field-types/knn-vector/) 名称的字段`passage_embedding`，其中包含文本的矢量嵌入。此外，将默认的摄入管道设置为`nlp-ingest-pipeline` 您在上一步中创建：


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
      "text": {
        "type": "text"
      }
    }
  }
}
```
{% include copy-curl.html %}

设置K-nn索引允许您以后执行矢量搜索`passage_embedding` 场地。

<详细信息关闭的markdown ="block">
  <summary>
    测试它
  </summary>
  {: .text-delta}

使用以下请求获取创建索引的设置和映射：

```json
GET /my-nlp-index/_settings
```
{% include copy-curl.html %}

```json
GET /my-nlp-index/_mappings
```
{% include copy-curl.html %}

</delect>

### 步骤2（c）：将文档摄入索引

在此步骤中，您将将几个示例文档摄入索引。示例数据取自[Flickr图像数据集](https://www.kaggle.com/datasets/hsankesara/flickr-image-dataset)。每个文档都包含一个`text` 与图像描述相对应的字段和`id` 与图像ID相对应的字段：

```json
PUT /my-nlp-index/_doc/1
{
  "text": "A West Virginia university women 's basketball team , officials , and a small gathering of fans are in a West Virginia arena .",
  "id": "4319130149.jpg"
}
```
{% include copy-curl.html %}

```json
PUT /my-nlp-index/_doc/2
{
  "text": "A wild animal races across an uncut field with a minimal amount of trees .",
  "id": "1775029934.jpg"
}
```
{% include copy-curl.html %}

```json
PUT /my-nlp-index/_doc/3
{
  "text": "People line the stands which advertise Freemont 's orthopedics , a cowboy rides a light brown bucking bronco .",
  "id": "2664027527.jpg"
}
```
{% include copy-curl.html %}

```json
PUT /my-nlp-index/_doc/4
{
  "text": "A man who is riding a wild horse in the rodeo is very near to falling off .",
  "id": "4427058951.jpg"
}
```
{% include copy-curl.html %}

```json
PUT /my-nlp-index/_doc/5
{
  "text": "A rodeo cowboy , wearing a cowboy hat , is being thrown off of a wild white horse .",
  "id": "2691147709.jpg"
}
```
{% include copy-curl.html %}

当文档摄入索引时`text_embedding` 处理器创建一个包含向量嵌入的附加字段，并将该字段添加到文档中。要查看索引的示例文档，请搜索文档1：

```json
GET /my-nlp-index/_doc/1
```
{% include copy-curl.html %}

响应包括文档`_source` 包含原件`text` 和`id` 字段和添加`passage_embedding` 场地：

```json
{
  "_index": "my-nlp-index",
  "_id": "1",
  "_version": 1,
  "_seq_no": 0,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "passage_embedding": [
      0.04491629,
      -0.34105563,
      0.036822468,
      -0.14139028,
      ...
    ],
    "text": "A West Virginia university women 's basketball team , officials , and a small gathering of fans are in a West Virginia arena .",
    "id": "4319130149.jpg"
  }
}
```

## 步骤3：搜索数据

现在，您将使用关键字搜索，神经搜索和两者组合搜索索引。

### 使用关键字搜索搜索

要使用关键字搜索搜索，请使用`match` 询问。您将从结果中排除嵌入：

```json
GET /my-nlp-index/_search
{
  "_source": {
    "excludes": [
      "passage_embedding"
    ]
  },
  "query": {
    "match": {
      "text": {
        "query": "wild west"
      }
    }
  }
}
```
{% include copy-curl.html %}

文档3未返回，因为它不包含指定的关键字。包含单词的文档`rodeo` 和`cowboy` 得分较低，因为不考虑语义含义：

<详细信息关闭的markdown ="block">
  <summary>
    结果
  </summary>
  {: .text-delta}

```json
{
  "took": 647,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4,
      "relation": "eq"
    },
    "max_score": 1.7878418,
    "hits": [
      {
        "_index": "my-nlp-index",
        "_id": "1",
        "_score": 1.7878418,
        "_source": {
          "text": "A West Virginia university women 's basketball team , officials , and a small gathering of fans are in a West Virginia arena .",
          "id": "4319130149.jpg"
        }
      },
      {
        "_index": "my-nlp-index",
        "_id": "2",
        "_score": 0.58093566,
        "_source": {
          "text": "A wild animal races across an uncut field with a minimal amount of trees .",
          "id": "1775029934.jpg"
        }
      },
      {
        "_index": "my-nlp-index",
        "_id": "5",
        "_score": 0.55228686,
        "_source": {
          "text": "A rodeo cowboy , wearing a cowboy hat , is being thrown off of a wild white horse .",
          "id": "2691147709.jpg"
        }
      },
      {
        "_index": "my-nlp-index",
        "_id": "4",
        "_score": 0.53899646,
        "_source": {
          "text": "A man who is riding a wild horse in the rodeo is very near to falling off .",
          "id": "4427058951.jpg"
        }
      }
    ]
  }
}
```
</details>

### 使用神经搜索搜索

要使用神经搜索搜索，请使用`neural` 查询并提供您之前设置的模型的模型ID，以便使用摄入时间使用的模型生成查询文本的向量嵌入：

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
        "query_text": "wild west",
        "model_id": "aVeif4oB5Vm0Tdw8zYO2",
        "k": 5
      }
    }
  }
}
```
{% include copy-curl.html %}

这次，响应不仅包含所有五个文档，而且文档顺序也得到了改善，因为神经搜索认为语义含义：

<详细信息关闭的markdown ="block">
  <summary>
    结果
  </summary>
  {: .text-delta}

```json
{
  "took": 25,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 5,
      "relation": "eq"
    },
    "max_score": 0.01585195,
    "hits": [
      {
        "_index": "my-nlp-index",
        "_id": "4",
        "_score": 0.01585195,
        "_source": {
          "text": "A man who is riding a wild horse in the rodeo is very near to falling off .",
          "id": "4427058951.jpg"
        }
      },
      {
        "_index": "my-nlp-index",
        "_id": "2",
        "_score": 0.015748845,
        "_source": {
          "text": "A wild animal races across an uncut field with a minimal amount of trees.",
          "id": "1775029934.jpg"
        }
      },
      {
        "_index": "my-nlp-index",
        "_id": "5",
        "_score": 0.015177963,
        "_source": {
          "text": "A rodeo cowboy , wearing a cowboy hat , is being thrown off of a wild white horse .",
          "id": "2691147709.jpg"
        }
      },
      {
        "_index": "my-nlp-index",
        "_id": "1",
        "_score": 0.013272902,
        "_source": {
          "text": "A West Virginia university women 's basketball team , officials , and a small gathering of fans are in a West Virginia arena .",
          "id": "4319130149.jpg"
        }
      },
      {
        "_index": "my-nlp-index",
        "_id": "3",
        "_score": 0.011347735,
        "_source": {
          "text": "People line the stands which advertise Freemont 's orthopedics , a cowboy rides a light brown bucking bronco .",
          "id": "2664027527.jpg"
        }
      }
    ]
  }
}
```
</details>

### 使用混合搜索搜索

混合搜索结合了关键字和神经搜索以提高搜索相关性。要实现混合搜索，您需要设置[搜索管道]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/index/) 在搜索时间运行。您将在中间阶段配置搜索搜索结果的搜索管道，并应用[`normalization-processor`]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/normalization-processor/) 给他们。这`normalization-processor` 将文档分数从多个查询子句中进行归一化并结合在一起，并根据所选的归一化和组合技术对文档进行撤销。

#### 步骤1：配置搜索管道

用一个配置搜索管道`normalization-processor`，使用以下请求。处理器中的标准化技术设置为`min_max`，组合技术设置为`arithmetic_mean`。这`weights` 数组将分配给每个查询子句分配的权重为十进制百分比：

```json
PUT /_search/pipeline/nlp-search-pipeline
{
  "description": "Post processor for hybrid search",
  "phase_results_processors": [
    {
      "normalization-processor": {
        "normalization": {
          "technique": "min_max"
        },
        "combination": {
          "technique": "arithmetic_mean",
          "parameters": {
            "weights": [
              0.3,
              0.7
            ]
          }
        }
      }
    }
  ]
}
```
{% include copy-curl.html %}

#### 步骤2：使用混合查询搜索

您将使用[`hybrid` 询问]({{site.url}}{{site.baseurl}}/query-dsl/compound/hybrid/) 结合`match` 和`neural` 查询条款。确保应用以前创建的`nlp-search-pipeline` 在查询参数中的请求：

```json
GET /my-nlp-index/_search?search_pipeline=nlp-search-pipeline
{
  "_source": {
    "exclude": [
      "passage_embedding"
    ]
  },
  "query": {
    "hybrid": {
      "queries": [
        {
          "match": {
            "text": {
              "query": "cowboy rodeo bronco"
            }
          }
        },
        {
          "neural": {
            "passage_embedding": {
              "query_text": "wild west",
              "model_id": "aVeif4oB5Vm0Tdw8zYO2",
              "k": 5
            }
          }
        }
      ]
    }
  }
}
```
{% include copy-curl.html %}

openSearch返回文档不仅与语义含义相匹配`wild west`，但是现在包含与野生西部主题有关的单词的文件也相对于其他单词的评分较高：

<详细信息关闭的markdown ="block">
  <summary>
    结果
  </summary>
  {: .text-delta}

```json
{
  "took": 27,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 5,
      "relation": "eq"
    },
    "max_score": 0.86481035,
    "hits": [
      {
        "_index": "my-nlp-index",
        "_id": "5",
        "_score": 0.86481035,
        "_source": {
          "text": "A rodeo cowboy , wearing a cowboy hat , is being thrown off of a wild white horse .",
          "id": "2691147709.jpg"
        }
      },
      {
        "_index": "my-nlp-index",
        "_id": "4",
        "_score": 0.7003,
        "_source": {
          "text": "A man who is riding a wild horse in the rodeo is very near to falling off .",
          "id": "4427058951.jpg"
        }
      },
      {
        "_index": "my-nlp-index",
        "_id": "2",
        "_score": 0.6839765,
        "_source": {
          "text": "A wild animal races across an uncut field with a minimal amount of trees.",
          "id": "1775029934.jpg"
        }
      },
      {
        "_index": "my-nlp-index",
        "_id": "3",
        "_score": 0.3007,
        "_source": {
          "text": "People line the stands which advertise Freemont 's orthopedics , a cowboy rides a light brown bucking bronco .",
          "id": "2664027527.jpg"
        }
      },
      {
        "_index": "my-nlp-index",
        "_id": "1",
        "_score": 0.29919013,
        "_source": {
          "text": "A West Virginia university women 's basketball team , officials , and a small gathering of fans are in a West Virginia arena .",
          "id": "4319130149.jpg"
        }
      }
    ]
  }
}
```
</details>

您没有在每个请求中指定搜索管道，而可以将其设置为索引的默认搜索管道，如下所示：

```json
PUT /my-nlp-index/_settings 
{
  "index.search.default_pipeline" : "nlp-search-pipeline"
}
```
{% include copy-curl.html %}

现在，您可以使用不同的权重，归一化技术和组合技术进行实验。有关更多信息，请参阅[`normalization-processor`]({{site.url}}{{site.baseurl}}/search-plugins/search-pipelines/normalization-processor/) 和[`hybrid` 询问]({{site.url}}{{site.baseurl}}/query-dsl/compound/hybrid/) 文档。

#### 先进的

您可以使用搜索模板来参数化搜索。搜索模板隐藏实现详细信息，减少嵌套级别的数量，从而减少查询复杂性。有关更多信息，请参阅[搜索模板]({{site.url}}{{site.baseurl}}/search-plugins/search-template/)。

### 清理

完成后，从集群中删除您在本教程中创建的组件：

```json
DELETE /my-nlp-index
```
{% include copy-curl.html %}

```json
DELETE /_search/pipeline/nlp-search-pipeline
```
{% include copy-curl.html %}

```json
DELETE /_ingest/pipeline/nlp-ingest-pipeline
```
{% include copy-curl.html %}

```json
POST /_plugins/_ml/models/aVeif4oB5Vm0Tdw8zYO2/_undeploy
```
{% include copy-curl.html %}

```json
DELETE /_plugins/_ml/models/aVeif4oB5Vm0Tdw8zYO2
```
{% include copy-curl.html %}

```json
DELETE /_plugins/_ml/model_groups/Z1eQf4oB5Vm0Tdw8EIP2
```
{% include copy-curl.html %}

## 进一步阅读

- 阅读有关OpenSearch语义搜索的基础知识[在OpenSearch中构建语义搜索引擎](https://opensearch.org/blog/semantic-search-solutions/)。
- 了解结合关键字和神经搜索，标准化和组合技术选项以及基准测试的好处[Opensearch中语义搜索的ABC：架构，基准和组合策略](https://opensearch.org/blog/semantic-science-benchmarks/)。


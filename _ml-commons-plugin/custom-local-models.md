---
layout: default
title: 自定义模型
parent: 在OpenSearch中使用ML模型
nav_order: 120
---

# 自定义本地型号
**通常可用2.9**
{: .label .label-purple }

要在本地使用自定义模型，您可以将其上传到OpenSearch集群。

## 模型支持

从OpenSearch 2.6开始，OpenSearch支持本地文本嵌入模型。

从OpenSearch 2.11开始，OpenSearch支持本地稀疏编码模型。

## 准备模型

对于文本嵌入和稀疏编码模型，您必须在模型zip文件中提供令牌json文件。

对于稀疏编码模型，请确保您的输出格式为`{"output":<sparse_vector>}` 以便ML Commons可以发布-处理稀疏矢量。

如果你很好-调整自己的数据集上的稀疏模型，您可能还需要使用自己的稀疏令牌模型。最好提供自己的[IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) 令牌模型zip文件中的JSON文件，因为当您在查询中使用令牌模型时，这会增加查询性能。另外，您可以使用OpenSearch-提供通用[来自MSMARCO的IDF](https://artifacts.opensearch.org/models/ml-models/amazon/neural-sparse/opensearch-neural-sparse-tokenizer-v1/1.0.0/torch_script/opensearch-neural-sparse-tokenizer-v1-1.0.0.zip)。如果未提供IDF文件，则将每个令牌的默认权重设置为1，这可能会影响稀疏的神经搜索性能。

### 模型格式

要在OpenSearch中使用模型，您需要将模型导出到便携式格式中。从2.5版开始，OpenSearch仅支持[Torchscript](https://pytorch.org/docs/stable/jit.html) 和[onnx](https://onnx.ai/) 格式。

在将其上传到OpenSearch之前，您必须将模型文件作为zip保存为zip。为确保ML Commons可以上传您的模型，请在上传之前压缩Torchscript文件。例如，下载火有关[型号文件](https://github.com/opensearch-project/ml-commons/blob/2.x/ml-algorithms/src/test/resources/org/opensearch/ml/engine/algorithms/text_embedding/all-MiniLM-L6-v2_torchscript_sentence-transformer.zip)。

此外，您必须为型号zip文件计算SHA256校验和在注册模型时需要提供的sha256校验和。例如，在UNIX上，使用以下命令获得校验和：

```bash
shasum -a 256 sentence-transformers_paraphrase-mpnet-base-v2-1.0.0-onnx.zip
```

### 型号大小

大多数深度学习模型都超过100 MB，因此很难将它们放入单个文档中。OpenSearch将模型文件拆分为较小的块，以存储在模型索引中。在为OpenSearch群集分配ML或数据节点时，请确保正确大小的ML节点，以便在进行ML推断时具有足够的内存。

## 先决条件

要将自定义模型上传到OpenSearch，您需要在OpenSearch集群之外准备它。您可以使用审计的模型，例如[拥抱脸](https://huggingface.co/)，或根据您的需求训练新模型。

### 集群设置

此示例使用一个没有专用ML节点的简单设置-ML节点。

在具有专用ML节点的群集上，指定`"only_run_on_ml_node": "true"` 以提高性能。有关更多信息，请参阅[ML Commons集群设置]({{site.url}}{{site.baseurl}}/ml-commons-plugin/cluster-settings/)。

为了确保此基本本地设置有效，请指定以下群集设置：

```json
PUT _cluster/settings
{
  "persistent": {
    "plugins": {
      "ml_commons": {
        "allow_registering_model_via_url": "true",
        "only_run_on_ml_node": "false",
        "model_access_control_enabled": "true",
        "native_memory_threshold": "99"
      }
    }
  }
}
```
{% include copy-curl.html %}

## 步骤1：注册模型组

要注册模型，您有以下选项：

- 您可以使用`model_group_id` 向现有模型组注册模型版本。
- 如果您不使用`model_group_id`，ML Commons使用新的模型组创建模型。

要注册模型组，请发送以下请求：

```json
POST /_plugins/_ml/model_groups/_register
{
  "name": "local_model_group",
  "description": "A model group for local models"
}
```
{% include copy-curl.html %}

响应包含您将用于将模型注册到此模型组的模型组ID：

```json
{
 "model_group_id": "wlcnb4kBJ1eYAeTMHlV6",
 "status": "CREATED"
}
```

要了解有关模型组的更多信息，请参阅[模型访问控制]({{site.url}}{{site.baseurl}}/ml-commons-plugin/model-access-control/)。

## 步骤2：注册本地模型

要向步骤1中创建的模型组注册远程模型，请在以下请求中从步骤1提供模型组ID：

```json
POST /_plugins/_ml/models/_register
{
  "name": "huggingface/sentence-transformers/msmarco-distilbert-base-tas-b",
  "version": "1.0.1",
  "model_group_id": "wlcnb4kBJ1eYAeTMHlV6",
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
  "created_time": 1676073973126,
  "url": "https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/msmarco-distilbert-base-tas-b/1.0.1/torch_script/sentence-transformers_msmarco-distilbert-base-tas-b-1.0.1-torch_script.zip"
}
```
{% include copy-curl.html %}

有关寄存器API参数的说明，请参阅[注册模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/model-apis/register-model/)。

OpenSearch返回寄存器操作的任务ID：

```json
{
  "task_id": "cVeMb4kBJ1eYAeTMFFgj",
  "status": "CREATED"
}
```

要检查操作的状态，将任务ID提供给[得到任务]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/tasks-apis/get-task/)：

```bash
GET /_plugins/_ml/tasks/cVeMb4kBJ1eYAeTMFFgj
```
{% include copy-curl.html %}

操作完成后，状态会更改为`COMPLETED`：

```json
{
  "model_id": "cleMb4kBJ1eYAeTMFFg4",
  "task_type": "REGISTER_MODEL",
  "function_name": "REMOTE",
  "state": "COMPLETED",
  "worker_node": [
    "XPcXLV7RQoi5m8NI_jEOVQ"
  ],
  "create_time": 1689793598499,
  "last_update_time": 1689793598530,
  "is_async": false
}
```

注意返回的`model_id` 因为您需要它来部署模型。

## 步骤3：部署模型

部署操作从模型索引中读取模型的块，然后创建一个模型的实例，以加载到内存中。模型越大，模型将模型加载到内存所需的时间越长。

要部署注册模型，请在以下请求中从步骤3提供其模型ID：

```bash
POST /_plugins/_ml/models/cleMb4kBJ1eYAeTMFFg4/_deploy
```
{% include copy-curl.html %}

响应包含您可以使用的任务ID来检查部署操作的状态：

```json
{
  "task_id": "vVePb4kBJ1eYAeTM7ljG",
  "status": "CREATED"
}
```

与上一步一样，通过调用任务API来检查操作的状态：

```bash
GET /_plugins/_ml/tasks/vVePb4kBJ1eYAeTM7ljG
```
{% include copy-curl.html %}

操作完成后，状态会更改为`COMPLETED`：

```json
{
  "model_id": "cleMb4kBJ1eYAeTMFFg4",
  "task_type": "DEPLOY_MODEL",
  "function_name": "REMOTE",
  "state": "COMPLETED",
  "worker_node": [
    "n-72khvBTBi3bnIIR8FTTw"
  ],
  "create_time": 1689793851077,
  "last_update_time": 1689793851101,
  "is_async": true
}
```

## 步骤4（可选）：测试模型

使用[预测API]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/predict/) 测试模型。

对于文本嵌入模型，请发送以下请求：

```json
POST /_plugins/_ml/_predict/text_embedding/cleMb4kBJ1eYAeTMFFg4
{
  "text_docs":[ "today is sunny"],
  "return_number": true,
  "target_response": ["sentence_embedding"]
}
```
{% include copy-curl.html %}

响应包含提供句子的文本嵌入：

```json
{
  "inference_results" : [
    {
      "output" : [
        {
          "name" : "sentence_embedding",
          "data_type" : "FLOAT32",
          "shape" : [
            768
          ],
          "data" : [
            0.25517133,
            -0.28009856,
            0.48519906,
            ...
          ]
        }
      ]
    }
  ]
}
```

对于稀疏编码模型，发送以下请求：

```json
POST /_plugins/_ml/_predict/sparse_encoding/cleMb4kBJ1eYAeTMFFg4
{
  "text_docs":[ "today is sunny"]
}
```
{% include copy-curl.html %}

响应包含令牌和权重：

```json
{
  "inference_results": [
    {
      "output": [
        {
          "name": "output",
          "dataAsMap": {
            "response": [
              {
                "saturday": 0.48336542,
                "week": 0.1034762,
                "mood": 0.09698499,
                "sunshine": 0.5738209,
                "bright": 0.1756877,
                ...
              }
          }
        }
    }
}
```

## 步骤5：使用模型进行搜索

要了解如何设置向量索引并使用文本嵌入模型进行搜索，请参见[神经文本搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-text-search/)。

要了解如何设置向量索引并使用稀疏编码模型进行搜索，请参见[神经稀疏搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-sparse-search/)。

要了解如何设置向量索引并使用多模式嵌入模型进行搜索，请参见[多模式搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-sparse-search/)。


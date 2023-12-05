---
layout: default
title: 预验证的模型
parent: 在OpenSearch中使用ML模型
nav_order: 120
---

# OpenSearch-提供了预告片的模型
**通常可用2.9**
{: .label .label-purple }

OpenSearch提供各种开放-源预处理的模型可以帮助一系列机器学习（ML）搜索和分析用例。您可以将任何支持的模型上传到OpenSearch集群并在本地使用。

## 先决条件

首先，选择其中之一[支持的预贴模型](#supported-pretrained-models)。

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

## 步骤2：注册本地OpenSearch-提供的模型

要向步骤1中创建的模型组注册远程模型，请在以下请求中从步骤1提供模型组ID。

因为预算**句子变压器** 模型起源于ML Commons模型存储库，您只需要提供`name`，`version`，`model_group_id`， 和`model_format` 在上传API请求中：

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

请注意**稀疏编码** 模型，您仍然需要上传完整的请求主体，如以下示例所示：

```json
POST /_plugins/_ml/models/_register
{
    "name": "amazon/neural-sparse/opensearch-neural-sparse-encoding-doc-v1",
    "version": "1.0.0",
    "model_group_id": "Z1eQf4oB5Vm0Tdw8EIP2",
    "description": "This is a neural sparse encoding model: It transfers text into sparse vector, and then extract nonzero index and value to entry and weights. It serves only in ingestion and customer should use tokenizer model in query.",
    "model_format": "TORCH_SCRIPT",
    "function_name": "SPARSE_ENCODING",
    "model_content_hash_value": "9a41adb6c13cf49a7e3eff91aef62ed5035487a6eca99c996156d25be2800a9a",
    "url":  "https://artifacts.opensearch.org/models/ml-models/amazon/neural-sparse/opensearch-neural-sparse-encoding-doc-v1/1.0.0/torch_script/opensearch-neural-sparse-encoding-doc-v1-1.0.0-torch_script.zip"
}
```
{% include copy-curl.html %}

你可以找到`url` 和`model_content_hash_value` 在每个模型的模型配置链接中。有关更多信息，请参阅[支持的预验证模型部分](#supported-pretrained-models)。设置`function_name` 到`SPARSE_ENCODING` 或者`SPARSE_TOKENIZE`。

请注意`function_name` 请求中的参数对应于`model_task_type` 模型配置中的参数。使用预告片的模型时，请确保从中更改参数的名称`model_task_type` 到`function_name` 在模型上传请求中。
{： 。重要的}

OpenSearch返回寄存器操作的任务ID：

```json
{
  "task_id": "cVeMb4kBJ1eYAeTMFFgj",
  "status": "CREATED"
}
```

要检查操作的状态，将任务ID提供给[任务API]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/tasks-apis/get-task/#get-a-task-by-id)：

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


## 支持的预贴模型

OpenSearch支持以下模型，按类型分类。文本嵌入模型来自[拥抱脸](https://huggingface.co/)。稀疏编码模型由Opensearch培训。尽管具有相同类型的模型将具有相似的用例，但每个模型的模型大小都不同，并且根据群集设置的性能不同。有关某些预算模型的性能比较，请参阅[Sbert文档](https://www.sbert.net/docs/pretrained_models.html#model-overview)。


### 句子变形金刚

句子变形金刚模型绘制句子和段落跨维密度向量空间的段落。向量的数量取决于模型的类型。您可以将这些模型用于用例，例如聚类或语义搜索。

下表提供了可以使用句子变压器模型和伪影链接的列表。请注意，您必须将模型名称与`huggingface/`，如图所示**型号名称** 柱子。

| 型号名称| 版本| 向量维度| 汽车-截断| 火炬伪影| Onnx工件|
|:---	|:---	|：---|:---	|:---	|
| `huggingface/sentence-transformers/all-distilroberta-v1` | 1.0.1| 768-尺寸密集的矢量空间。| 是的| - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/all-distilroberta-v1/1.0.1/torch_script/sentence-transformers_all-distilroberta-v1-1.0.1-torch_script.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/all-distilroberta-v1/1.0.1/torch_script/config.json) | - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/all-distilroberta-v1/1.0.1/onnx/sentence-transformers_all-distilroberta-v1-1.0.1-onnx.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/all-distilroberta-v1/1.0.1/onnx/config.json) |
| `huggingface/sentence-transformers/all-MiniLM-L6-v2` | 1.0.1| 384-尺寸密集的矢量空间。| 是的| - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/all-MiniLM-L6-v2/1.0.1/torch_script/sentence-transformers_all-MiniLM-L6-v2-1.0.1-torch_script.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/all-MiniLM-L6-v2/1.0.1/torch_script/config.json) | - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/all-MiniLM-L6-v2/1.0.1/onnx/sentence-transformers_all-MiniLM-L6-v2-1.0.1-onnx.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/all-MiniLM-L6-v2/1.0.1/onnx/config.json) |
| `huggingface/sentence-transformers/all-MiniLM-L12-v2` | 1.0.1| 384-尺寸密集的矢量空间。| 是的| - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/all-MiniLM-L12-v2/1.0.1/torch_script/sentence-transformers_all-MiniLM-L12-v2-1.0.1-torch_script.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/all-MiniLM-L12-v2/1.0.1/onnx/config.json) | - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/all-MiniLM-L12-v2/1.0.1/onnx/sentence-transformers_all-MiniLM-L12-v2-1.0.1-onnx.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/all-MiniLM-L12-v2/1.0.1/onnx/config.json) |
| `huggingface/sentence-transformers/all-mpnet-base-v2` | 1.0.1| 768-尺寸密集的矢量空间。| 是的| - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/all-mpnet-base-v2/1.0.1/torch_script/sentence-transformers_all-mpnet-base-v2-1.0.1-torch_script.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/all-mpnet-base-v2/1.0.1/torch_script/config.json) | - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/all-mpnet-base-v2/1.0.1/onnx/sentence-transformers_all-mpnet-base-v2-1.0.1-onnx.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/all-mpnet-base-v2/1.0.1/onnx/config.json) |
| `huggingface/sentence-transformers/msmarco-distilbert-base-tas-b` | 1.0.1| 768-尺寸密集的矢量空间。优化用于语义搜索。| 不| - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/msmarco-distilbert-base-tas-b/1.0.1/torch_script/sentence-transformers_msmarco-distilbert-base-tas-b-1.0.1-torch_script.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/msmarco-distilbert-base-tas-b/1.0.1/torch_script/config.json) | - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/msmarco-distilbert-base-tas-b/1.0.1/onnx/sentence-transformers_msmarco-distilbert-base-tas-b-1.0.1-onnx.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/msmarco-distilbert-base-tas-b/1.0.1/onnx/config.json) |
| `huggingface/sentence-transformers/multi-qa-MiniLM-L6-cos-v1` | 1.0.1| 384-尺寸密集的矢量空间。专为语义搜索而设计，并通过2.15亿个问答对培训。| 是的| - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/multi-qa-MiniLM-L6-cos-v1/1.0.1/torch_script/sentence-transformers_multi-qa-MiniLM-L6-cos-v1-1.0.1-torch_script.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/multi-qa-MiniLM-L6-cos-v1/1.0.1/torch_script/config.json) | - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/multi-qa-MiniLM-L6-cos-v1/1.0.1/onnx/sentence-transformers_multi-qa-MiniLM-L6-cos-v1-1.0.1-onnx.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/multi-qa-MiniLM-L6-cos-v1/1.0.1/onnx/config.json) |
| `huggingface/sentence-transformers/multi-qa-mpnet-base-dot-v1` | 1.0.1| 384-尺寸密集的矢量空间。| 是的| - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/multi-qa-mpnet-base-dot-v1/1.0.1/torch_script/sentence-transformers_multi-qa-mpnet-base-dot-v1-1.0.1-torch_script.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/multi-qa-mpnet-base-dot-v1/1.0.1/torch_script/config.json) | - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/multi-qa-mpnet-base-dot-v1/1.0.1/onnx/sentence-transformers_multi-qa-mpnet-base-dot-v1-1.0.1-onnx.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/multi-qa-mpnet-base-dot-v1/1.0.1/onnx/config.json) |
| `huggingface/sentence-transformers/paraphrase-MiniLM-L3-v2` | 1.0.1| 384-尺寸密集的矢量空间。| 是的| - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/paraphrase-MiniLM-L3-v2/1.0.1/torch_script/sentence-transformers_paraphrase-MiniLM-L3-v2-1.0.1-torch_script.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/paraphrase-MiniLM-L3-v2/1.0.1/torch_script/config.json) | - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/paraphrase-MiniLM-L3-v2/1.0.1/onnx/sentence-transformers_paraphrase-MiniLM-L3-v2-1.0.1-onnx.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/paraphrase-MiniLM-L3-v2/1.0.1/onnx/config.json) |
| `huggingface/sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2` | 1.0.1| 384-尺寸密集的矢量空间。| 是的| - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2/1.0.1/torch_script/sentence-transformers_paraphrase-multilingual-MiniLM-L12-v2-1.0.1-torch_script.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2/1.0.1/torch_script/config.json) | - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2/1.0.1/onnx/sentence-transformers_paraphrase-multilingual-MiniLM-L12-v2-1.0.1-onnx.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2/1.0.1/onnx/config.json) |
| `huggingface/sentence-transformers/paraphrase-mpnet-base-v2` | 1.0.0| 768-尺寸密集的矢量空间。| 是的| - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/paraphrase-mpnet-base-v2/1.0.0/torch_script/sentence-transformers_paraphrase-mpnet-base-v2-1.0.0-torch_script.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/paraphrase-mpnet-base-v2/1.0.0/torch_script/config.json) | - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/paraphrase-mpnet-base-v2/1.0.0/onnx/sentence-transformers_paraphrase-mpnet-base-v2-1.0.0-onnx.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/paraphrase-mpnet-base-v2/1.0.0/onnx/config.json) |
| `huggingface/sentence-transformers/distiluse-base-multilingual-cased-v1` | 1.0.1| 512-尺寸密集的矢量空间。| 是的| - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/distiluse-base-multilingual-cased-v1/1.0.1/torch_script/sentence-transformers_distiluse-base-multilingual-cased-v1-1.0.1-torch_script.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/distiluse-base-multilingual-cased-v1/1.0.1/torch_script/config.json) | 无法使用|
| `huggingface/sentence-transformers/paraphrase-mpnet-base-v2` | 1.0.0| 768-尺寸密集的矢量空间。| 是的| - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/paraphrase-mpnet-base-v2/1.0.0/torch_script/sentence-transformers_paraphrase-mpnet-base-v2-1.0.0-torch_script.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/paraphrase-mpnet-base-v2/1.0.0/torch_script/config.json) | - [model_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/paraphrase-mpnet-base-v2/1.0.0/onnx/sentence-transformers_paraphrase-mpnet-base-v2-1.0.0-onnx.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/paraphrase-mpnet-base-v2/1.0.0/onnx/config.json) |

### 稀疏编码模型
**引入2.11**
{: .label .label-purple }

稀疏编码模型将文本传输到稀疏向量，然后将向量转换为列表`<token: weight>` 对代表文本条目及其在稀疏矢量中的相应权重。您可以将这些模型用于用例，例如聚类或稀疏神经搜索。

我们建议使用以下模型以进行最佳性能：

- 使用`amazon/neural-sparse/opensearch-neural-sparse-encoding-v1` 摄入和搜索过程中的模型。
- 使用`amazon/neural-sparse/opensearch-neural-sparse-encoding-doc-v1` 摄入期间的模型
`amazon/neural-sparse/opensearch-neural-sparse-tokenizer-v1` 搜索过程中的模型。

下表提供了可用于下载它们的稀疏编码模型和工件链接的列表。

| 型号名称| 汽车-截断| 火炬伪影| 描述|
|---|---|---|
| `amazon/neural-sparse/opensearch-neural-sparse-encoding-v1` | 是的| - [model_url](https://artifacts.opensearch.org/models/ml-models/amazon/neural-sparse/opensearch-neural-sparse-encoding-v1/1.0.0/torch_script/opensearch-neural-sparse-encoding-v1-1.0.0-torch_script.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/amazon/neural-sparse/opensearch-neural-sparse-encoding-v1/1.0.0/torch_script/config.json) | 神经稀疏编码模型。该模型将文本转换为稀疏的向量，确定了非索引-向量中的零元素，然后将向量转换为`<entry, weight>` 对，每个条目对应于非-零元素索引。|
| `amazon/neural-sparse/opensearch-neural-sparse-encoding-doc-v1` | 是的| - [model_url](https://artifacts.opensearch.org/models/ml-models/amazon/neural-sparse/opensearch-neural-sparse-encoding-doc-v1/1.0.0/torch_script/opensearch-neural-sparse-encoding-doc-v1-1.0.0-torch_script.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/amazon/neural-sparse/opensearch-neural-sparse-encoding-doc-v1/1.0.0/torch_script/config.json) | 神经稀疏编码模型。该模型将文本转换为稀疏的向量，确定了非索引-向量中的零元素，然后将向量转换为`<entry, weight>` 对，每个条目对应于非-零元素索引。|
| `amazon/neural-sparse/opensearch-neural-sparse-tokenizer-v1` | 是的| - [model_url](https://artifacts.opensearch.org/models/ml-models/amazon/neural-sparse/opensearch-neural-sparse-tokenizer-v1/1.0.0/torch_script/opensearch-neural-sparse-tokenizer-v1-1.0.0.zip)<br>- [config_url](https://artifacts.opensearch.org/models/ml-models/amazon/neural-sparse/opensearch-neural-sparse-tokenizer-v1/1.0.0/torch_script/config.json) | 神经稀疏令牌模型。该模型将文本将文本定为令牌，并分配每个令牌预定义的权重，即代币的IDF（如果未提供IDF文件，则权重默认为1）。有关更多信息，请参阅[准备模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/custom-local-models/#preparing-a-model)。


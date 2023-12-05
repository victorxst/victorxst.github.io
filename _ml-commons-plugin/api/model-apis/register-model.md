---
layout: default
title: 注册模型
parent: 模型API
grand_parent: ML Commons API
nav_order: 10
---

# 注册模型

特定模型的所有版本均在模型组中保存。你也可以[注册模型组]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/model-group-apis/register-model-group/) 在将模型注册到组或注册模型的第一个版本之前，请创建组。集群中的每个模型组名称必须在全球范围内唯一。

如果您在不首先注册模型组的情况下注册模型的第一个版本，则将自动创建一个新的模型组，并使用以下名称和访问级别：

- 名称：新型号组将与模型具有相同的名称。因为模型组名称必须是唯一的，请确保您的型号名称与群集中的任何模型组没有相同的名称。
- 访问级别：新模型组的访问级别是使用`access_mode`，`backend_roles`， 和`add_all_backend_roles` 您在请求中传递的参数。如果您没有提供这三个参数，那么新模型组将是`private` 如果在群集上启用了模型访问控制`public` 如果禁用了模型访问控制。新注册的模型是分配给该模型组的第一个模型版本。

创建模型组后，提供其`model_group_id` 向模型组注册新的型号版本。在这种情况下，模型名称不需要唯一。

如果您正在使用[预验证的模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/pretrained-models#supported-pretrained-models) 由OpenSearch提供，我们建议您首先注册一个模型组，其中唯一名称为这些型号。然后注册验证的模型作为该模型组的版本。这样可以确保每个模型组都有一个全球唯一的模型组名称。
{: .tip}

有关此API的用户访问的信息，请参见[模型访问控制注意事项]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/model-apis/index/#model-access-control-considerations)。

如果模型的尺寸超过10 MB，则ML Commons将其分为较小的块，并将这些块保存在模型的索引中。

## 路径和HTTP方法

```json
POST /_plugins/_ml/models/_register
```
{% include copy-curl.html %}

## 注册opensearch-提供了预验证的模型

OpenSearch提供了几种审慎的模型。有关更多信息，请参阅[OpenSearch-提供了预告片的模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/pretrained-models/)。

### 注册据审计的文本嵌入模型

要注册预验证的文本嵌入模型，唯一需要的参数是`name`，`version`， 和`model_format`。

#### 请求字段

下表列出了可用的请求字段。

场地| 数据类型| 必需/可选| 描述
：---  | :--- | :--- 
`name`| 细绳| 必需的| 模型名称。|
`version` | 整数| 必需的| 型号版本号。|
`model_format` | 细绳| 必需的| 模型文件的便携式格式。有效值是`TORCH_SCRIPT` 和`ONNX`。|
`description` | 细绳| 选修的| 模型描述。|
`model_group_id` | 细绳| 选修的| 模型组的模型组ID将此模型注册为。

#### 示例请求：OpenSearch-提供文本嵌入模型

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

### 注册预验证的稀疏编码模型

要注册预估计的稀疏编码模型，您必须将功能名称设置为`SPARSE_ENCODING` 或者`SPARSE_TOKENIZE`。

#### 请求字段

下表列出了可用的请求字段。

场地| 数据类型| 必需/可选| 描述
：---  | :--- | :--- 
`name`| 细绳| 必需的| 模型名称。|
`version` | 整数| 必需的| 型号版本号。|
`model_format` | 细绳| 必需的| 模型文件的便携式格式。有效值是`TORCH_SCRIPT` 和`ONNX`。|
`function_name` | 细绳| 必需的| 将此参数设置为`SPARSE_ENCODING` 或者`SPARSE_TOKENIZE`。
`model_content_hash_value` | 细绳| 必需的| 使用SHA生成的模型内容哈希-256哈希算法。
`url` | 细绳| 必需的| 包含模型的URL。|
`description` | 细绳| 选修的| 模型描述。|
`model_group_id` | 细绳| 选修的| 模型组的模型组ID将此模型注册为。

#### 示例请求：OpenSearch-提供了稀疏编码模型

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

## 注册自定义模型

要在OpenSearch Cluster中本地使用自定义模型，您需要为该模型提供URL和配置对象。有关更多信息，请参阅[自定义本地型号]({{site.url}}{{site.baseurl}}/ml-commons-plugin/custom-local-models/)。

### 请求字段

下表列出了可用的请求字段。

场地| 数据类型| 必需/可选| 描述
：---  | :--- | :--- 
`name`| 细绳| 必需的| 模型名称。|
`version` | 整数| 必需的| 型号版本号。|
`model_format` | 细绳| 必需的| 模型文件的便携式格式。有效值是`TORCH_SCRIPT` 和`ONNX`。|
`function_name` | 细绳| 必需的| 将此参数设置为`SPARSE_ENCODING` 或者`SPARSE_TOKENIZE`。
`model_content_hash_value` | 细绳| 必需的| 使用SHA生成的模型内容哈希-256哈希算法。
[`model_config`](#the-model_config-object)  | 目的| 必需的| 模型的配置，包括`model_type`，`embedding_dimension`， 和`framework_type`。`all_config` 是一个可选的JSON字符串，包含所有模型配置。|
`url` | 细绳| 必需的| 包含模型的URL。|
`description` | 细绳| 选修的| 模型描述。|
`model_group_id` | 细绳| 选修的| 模型组的模型组ID将此模型注册为。

#### 这`model_config` 目的

| 场地| 数据类型| 描述|
| :--- | :--- | :--- 
| `model_type` | 细绳| 模型类型，例如`bert`。对于拥抱的面部模型，模型类型已在`config.json`。例如，请参见[`all-MiniLM-L6-v2` 拥抱面部模型`config.json`](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2/blob/main/config.json#L15)。必需的。|
| `embedding_dimension` | 整数| 模型的维度-生成密集的矢量。对于拥抱面部模型，尺寸是在模型卡中指定的。例如，在[`all-MiniLM-L6-v2` 拥抱面部型号卡](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2)， 该声明`384 dimensional dense vector space` 将384指定为嵌入尺寸。必需的。|
| `framework_type` | 细绳| 模型正在使用的框架。目前，OpenSearch支持`sentence_transformers` 和`huggingface_transformers` 构架。这`sentence_transformers` 模型输出文本嵌入直接，因此ML Commons不会执行任何后处理。为了`huggingface_transformers`，ML Commons通过应用平均合并以获取文本嵌入来执行后处理。请参阅示例[`all-MiniLM-L6-v2` 拥抱面部模型](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2) 更多细节。必需的。|
| `all_config` | 细绳| 该字段用于参考目的。您可以在此字段中指定所有模型配置。例如，如果您使用的是拥抱面部模型，则可以缩小`config.json` 文件到一行，并将其内容保存在`all_config` 场地。上传模型后，您可以使用GET模型API操作来获取该字段中存储的所有模型配置。选修的。|

您可以进一步自定义审慎的句子变压器模型的帖子-处理以下可选字段的处理逻辑`model_config` 目的。

| 场地| 数据类型| 描述|
| :--- | :--- | :--- |
| `pooling_mode` | 细绳| 帖子-过程模型输出，要么`mean`，`mean_sqrt_len`，`max`，`weightedmean`， 或者`cls`。|
| `normalize_result` | 布尔| 设置为`true`，使模型输出归一化，以扩展到模型的标准范围。|

#### 示例请求：自定义模型

以下示例请求登记一个版本`1.0.0` NLP句子转换模型的名称`all-MiniLM-L6-v2`。

```json
POST /_plugins/_ml/models/_register
{
    "name": "all-MiniLM-L6-v2",
    "version": "1.0.0",
    "description": "test model",
    "model_format": "TORCH_SCRIPT",
    "model_group_id": "FTNlQ4gBYW0Qyy5ZoxfR",
    "model_content_hash_value": "c15f0d2e62d872be5b5bc6c84d2e0f4921541e29fefbef51d59cc10a8ae30e0f",
    "model_config": {
        "model_type": "bert",
        "embedding_dimension": 384,
        "framework_type": "sentence_transformers",
       "all_config": "{\"_name_or_path\":\"nreimers/MiniLM-L6-H384-uncased\",\"architectures\":[\"BertModel\"],\"attention_probs_dropout_prob\":0.1,\"gradient_checkpointing\":false,\"hidden_act\":\"gelu\",\"hidden_dropout_prob\":0.1,\"hidden_size\":384,\"initializer_range\":0.02,\"intermediate_size\":1536,\"layer_norm_eps\":1e-12,\"max_position_embeddings\":512,\"model_type\":\"bert\",\"num_attention_heads\":12,\"num_hidden_layers\":6,\"pad_token_id\":0,\"position_embedding_type\":\"absolute\",\"transformers_version\":\"4.8.2\",\"type_vocab_size\":2,\"use_cache\":true,\"vocab_size\":30522}"
    },
    "url": "https://artifacts.opensearch.org/models/ml-models/huggingface/sentence-transformers/all-MiniLM-L6-v2/1.0.1/torch_script/sentence-transformers_all-MiniLM-L6-v2-1.0.1-torch_script.zip"
}
```
{% include copy-curl.html %}

## 注册在第三个托管的模型-派对平台

注册在第三个托管的模型-派对平台，您可以首先创建一个独立的连接器，并提供该连接器的ID，也可以为模型指定内部连接器。有关更多信息，请参阅[创建第三个连接器-政党ML平台]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/connectors/)。

### 请求字段

下表列出了可用的请求字段。

场地| 数据类型| 必需/可选| 描述
：---  | :--- | :--- 
`name`| 细绳| 必需的| 模型名称。|
`function_name` | 细绳| 必需的| 将此参数设置为`SPARSE_ENCODING` 或者`SPARSE_TOKENIZE`。
`connector_id` | 选修的| 必需的| 独立连接器的连接器ID与托管在第三个型号的型号-派对平台。有关更多信息，请参阅[独立连接器]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/connectors/#standalone-connector)。您必须提供任何`connector_id` 或者`connector`。
`connector` | 目的| 必需的| 包含内部连接器的规格，该连接器与第三个模型托管的模型-派对平台。有关更多信息，请参阅[内部连接器]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/connectors/#internal-connector)。您必须提供任何`connector_id` 或者`connector`。
`description` | 细绳| 选修的| 模型描述。|
`model_group_id` | 细绳| 选修的| 模型组的模型组ID将此模型注册为。

#### 示例请求：带独立连接器的远程模型

```json
POST /_plugins/_ml/models/_register
{
    "name": "openAI-gpt-3.5-turbo",
    "function_name": "remote",
    "model_group_id": "1jriBYsBq7EKuKzZX131",
    "description": "test model",
    "connector_id": "a1eMb4kBJ1eYAeTMAljY"
}
```
{% include copy-curl.html %}

#### 示例请求：带有内部连接器的远程模型

```json
POST /_plugins/_ml/models/_register
{
    "name": "openAI-GPT-3.5: internal connector",
    "function_name": "remote",
    "model_group_id": "lEFGL4kB4ubqQRzegPo2",
    "description": "test model",
    "connector": {
        "name": "OpenAI Connector",
        "description": "The connector to public OpenAI model service for GPT 3.5",
        "version": 1,
        "protocol": "http",
        "parameters": {
            "endpoint": "api.openai.com",
            "max_tokens": 7,
            "temperature": 0,
            "model": "text-davinci-003"
        },
        "credential": {
            "openAI_key": "..."
        },
        "actions": [
            {
                "action_type": "predict",
                "method": "POST",
                "url": "https://${parameters.endpoint}/v1/completions",
                "headers": {
                    "Authorization": "Bearer ${credential.openAI_key}"
                },
                "request_body": "{ \"model\": \"${parameters.model}\", \"prompt\": \"${parameters.prompt}\", \"max_tokens\": ${parameters.max_tokens}, \"temperature\": ${parameters.temperature} }"
            }
        ]
    }
}
```
{% include copy-curl.html %}

#### 示例响应

OpenSearch用`task_id` 和任务`status`。

```json
{
  "task_id" : "ew8I44MBhyWuIwnfvDIH", 
  "status" : "CREATED"
}
```

## 检查模型注册的状态

要查看模型注册的状态并检索为新模型版本创建的模型ID，请通过`task_id` 作为任务API的路径参数：

```json
GET /_plugins/_ml/tasks/<task_id>
```
{% include copy-curl.html %}

响应包含模型版本的模型ID：

```json
{
  "model_id": "Qr1YbogBYOqeeqR7sI9L",
  "task_type": "DEPLOY_MODEL",
  "function_name": "TEXT_EMBEDDING",
  "state": "COMPLETED",
  "worker_node": [
    "N77RInqjTSq_UaLh1k0BUg"
  ],
  "create_time": 1685478486057,
  "last_update_time": 1685478491090,
  "is_async": true
}
```


---
layout: default
title: 连接到远程模型
has_children: true
has_toc: false
nav_order: 60
---

# 连接到远程型号
**引入2.9**
{: .label .label-purple }

机器学习（ML）可扩展性使ML开发人员能够与其他ML服务（例如Amazon Sagemaker或OpenAI）建立集成。这些集成使系统管理员和数据科学家能够在其OpenSearch集群之外运行ML工作负载。

要开始使用ML可扩展性，请从以下选项中选择：

- 如果您是ML的开发人员，希望与您的特定ML服务集成，请参见[连接器蓝图]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/blueprints/)。
- 如果您是系统管理员或数据科学家，希望创建与ML服务的连接，请参见[连接器]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/connectors/)。

## 先决条件

如果您是部署ML连接器的管理员，请确保连接器的目标模型已在您选择的平台上部署。此外，请确保您有权将数据发送和接收到第三-连接器的派对API。

当您的第三次启用访问控制时-派对平台，您可以使用`authorization` 或者`credential` 连接器API中的设置。

### 添加可信赖的端点

要配置OpenSearch中的连接器`plugins.ml_commons.trusted_connector_endpoints_regex` 设置，支持Java Regex表达式：

```json
PUT /_cluster/settings
{
    "persistent": {
        "plugins.ml_commons.trusted_connector_endpoints_regex": [
          "^https://runtime\\.sagemaker\\..*[a-z0-9-]\\.amazonaws\\.com/.*$",
          "^https://api\\.openai\\.com/.*$",
          "^https://api\\.cohere\\.ai/.*$",
          "^https://bedrock-runtime\\..*[a-z0-9-]\\.amazonaws\\.com/.*$"
        ]
    }
}
```
{% include copy-curl.html %}



### 设置连接器访问控制

如果您打算使用远程连接器，请确保使用启用安全插件的OpenSearch集群。使用安全插件可让您访问连接器访问控制，这是使用远程连接器时所需的。
{: .warning}

如果您需要连接器的粒度访问控件，请使用以下群集设置：

```json
PUT /_cluster/settings
{
    "persistent": {
        "plugins.ml_commons.connector_access_control_enabled": true
    }
}
```
{% include copy-curl.html %}

启用访问控制时，您可以安装[安全插件]({{site.url}}{{site.baseurl}}/security/index/)。这使得`backend_roles`，`add_all_backend_roles`， 或者`access_model` 为了使用连接器API所需的选项。如果成功，OpenSearch返回以下回复：

```json
{
  "acknowledged": true,
  "persistent": {
    "plugins": {
      "ml_commons": {
        "connector_access_control_enabled": "true"
      }
    }
  },
  "transient": {}
}
```

### 节点设置

基于外部连接器的远程模型消耗的资源较少。因此，您可以使用数据节点从独立连接器中部署任何模型。为了确保您的独立连接使用数据节点，请设置`plugins.ml_commons.only_run_on_ml_node` 到`false`：

```json
PUT /_cluster/settings
{
    "persistent": {
        "plugins.ml_commons.only_run_on_ml_node": false
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
  "name": "remote_model_group",
  "description": "A model group for remote models"
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

## 步骤2：创建一个连接器

您可以作为特定模型的一部分创建独立的连接器或内部连接器。有关连接器和连接器示例的更多信息，请参见[连接器]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/connectors/)。

连接器创建API，`/_plugins/_ml/connectors/_create`，创建连接器，以促进OpenSearch中的注册和部署外部模型。使用`endpoint` 参数，您可以使用其特定的API端点将ML Commons连接到任何受支持的ML工具。例如，您可以使用`api.openai.com` 端点：

```json
POST /_plugins/_ml/connectors/_create
{
    "name": "OpenAI Chat Connector",
    "description": "The connector to public OpenAI model service for GPT 3.5",
    "version": 1,
    "protocol": "http",
    "parameters": {
        "endpoint": "api.openai.com",
        "model": "gpt-3.5-turbo"
    },
    "credential": {
        "openAI_key": "..."
    },
    "actions": [
        {
            "action_type": "predict",
            "method": "POST",
            "url": "https://${parameters.endpoint}/v1/chat/completions",
            "headers": {
                "Authorization": "Bearer ${credential.openAI_key}"
            },
            "request_body": "{ \"model\": \"${parameters.model}\", \"messages\": ${parameters.messages} }"
        }
    ]
}
```
{% include copy-curl.html %}

响应包含新创建的连接器的连接器ID：

```json
{
  "connector_id": "a1eMb4kBJ1eYAeTMAljY"
}
```

## 步骤3：注册远程模型

要向步骤1中创建的模型组注册远程模型，请在以下请求中从步骤1提供模型组ID，并从步骤2提供连接器ID：

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

## 步骤4：部署远程模型

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

## 步骤5（可选）：测试远程模型

使用[预测API]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/predict/) 测试模型：

```json
POST /_plugins/_ml/models/cleMb4kBJ1eYAeTMFFg4/_predict
{
  "parameters": {
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful assistant."
      },
      {
        "role": "user",
        "content": "Hello!"
      }
    ]
  }
}
```
{% include copy-curl.html %}

要了解OpenAI中有关聊天功能的更多信息，请参阅[Openai Chat API](https://platform.openai.com/docs/api-reference/chat)。

响应包含OpenAI模型提供的推论结果：

```json
{
  "inference_results": [
    {
      "output": [
        {
          "name": "response",
          "dataAsMap": {
            "id": "chatcmpl-7e6s5DYEutmM677UZokF9eH40dIY7",
            "object": "chat.completion",
            "created": 1689793889,
            "model": "gpt-3.5-turbo-0613",
            "choices": [
              {
                "index": 0,
                "message": {
                  "role": "assistant",
                  "content": "Hello! How can I assist you today?"
                },
                "finish_reason": "stop"
              }
            ],
            "usage": {
              "prompt_tokens": 19,
              "completion_tokens": 9,
              "total_tokens": 28
            }
          }
        }
      ]
    }
  ]
}
```

## 步骤6：使用模型进行搜索

要了解如何设置向量索引并使用文本嵌入模型进行搜索，请参见[神经文本搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-text-search/)。

要了解如何设置向量索引并使用稀疏编码模型进行搜索，请参见[神经稀疏搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-sparse-search/)。

要了解如何设置向量索引并使用多模式嵌入模型进行搜索，请参见[多模式搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-sparse-search/)。

## 下一步

- 有关连接器的更多信息，包括连接器示例，请参阅[连接器]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/connectors/)。
- 有关连接器参数的更多信息，请参阅[连接器蓝图]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/blueprints/)。
- 有关在OpenSearch中与ML模型互动的更多信息，请参见[在OpenSearch仪表板中管理ML模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-dashboard/)


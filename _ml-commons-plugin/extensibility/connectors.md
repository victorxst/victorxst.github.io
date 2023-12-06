---
layout: default
title: 连接器
has_children: false
has_toc: false
nav_order: 61
parent: 连接到远程模型
---

# 创建第三个连接器-政党ML平台
**引入2.9**
{: .label .label-purple }

连接器有助于访问第三托托的远程模型-派对平台。

您可以通过两种方式提供连接器：

1. 创建一个[独立连接器](#standalone-connector)：可以通过多个远程模型重复使用并共享独立的连接器，但需要访问OpenSearch中的模型和连接器和第三个连接器-派对平台（例如OpenAI或Amazon Sagemaker）正在访问连接器。独立连接器保存在连接器索引中。

2. 使用一个远程模型[内部连接器](#internal-connector)：只能与创建其创建的远程模型一起使用内部连接器。要访问内部连接器，您只需要访问模型本身，因为该连接是在模型内建立的。内部连接器保存在模型索引中。

## 支持的连接器

从OpenSearch 2.9开始，已经对连接器进行了以下ML服务的测试，尽管可以为此处未列出的其他平台创建连接器：

- [亚马逊射手制造商](https://aws.amazon.com/sagemaker/) 允许您托管和管理文本嵌入模型的生命周期，并在OpenSearch中为语义搜索查询提供动力。连接后，Amazon Sagemaker托管您的模型，OpenSearch用于查询推断。这使重视其功能的Amazon Sagemaker用户（例如模型监视，无服务器托管以及用于连续培训和部署的工作流程自动化）。
- [Openai Chatgpt](https://openai.com/blog/chatgpt) 使您可以从OpenSearch集群中调用OpenAI聊天模型。
- [共同](https://cohere.com/) 允许您使用OpenSearch的数据为Cohere大型语言模型供电。
- 这[基岩泰坦嵌入](https://aws.amazon.com/bedrock/titan/) 模型可以推动语义搜索和检索-OpenSearch中的增强产生。

所有连接器均由机器学习（ML）开发人员创建的JSON蓝图组成。蓝图允许管理员和数据科学家在OpenSearch和AI服务或模型之间建立联系-服务技术。

您可以在每个连接器中找到蓝图[ML Commons存储库](https://github.com/opensearch-project/ml-commons/tree/2.x/docs/remote_inference_blueprints)。

有关蓝图参数的更多信息，请参见[连接器蓝图]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/blueprints/)。

管理员只需要输入他们的`credential` 设置，例如`"openAI_key"`，对于他们正在连接的服务。所有其他参数均在[蓝图]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/blueprints/)。
{:.note}

## 独立连接器

要创建独立的连接器，请将请求发送到`connectors/_create` 端点并提供所有描述的参数[连接器蓝图]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/blueprints/)：

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

## 内部连接器

要创建内部连接器，请提供所有描述的参数[连接器蓝图]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/blueprints/) 在`connector` 请求的对象`models/_register` 端点：

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

## OpenAI聊天连接器

以下示例创建了独立的OpenAI聊天连接器。可以将相同的选项用于内部连接器下的内部连接器`connector` 范围：


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

创建连接器后，您可以检索`task_id` 和`connector_id` 注册和部署模型，然后使用预测API，类似于独立的连接器。


## Amazon Sagemaker连接器

以下示例创建了独立的Amazon Sagemaker连接器。可以将相同的选项用于内部连接器下的内部连接器`connector` 范围：

```json
POST /_plugins/_ml/connectors/_create
{
    "name": "sagemaker: embedding",
    "description": "Test connector for Sagemaker embedding model",
    "version": 1,
    "protocol": "aws_sigv4",
    "credential": {
        "access_key": "...",
        "secret_key": "...",
        "session_token": "..."
    },
    "parameters": {
        "region": "us-west-2",
        "service_name": "sagemaker"
    },
    "actions": [
        {
            "action_type": "predict",
            "method": "POST",
            "headers": {
                "content-type": "application/json"
            },
            "url": "https://runtime.sagemaker.${parameters.region}.amazonaws.com/endpoints/lmi-model-2023-06-24-01-35-32-275/invocations",
            "request_body": "[\"${parameters.inputs}\"]"
        }
    ]
}
```

这`credential` 参数包含以下选项保留的选项`aws_sigv4` 验证：

- `access_key`： 必需的。提供AWS实例的访问密钥。
- `secret_key`： 必需的。为AWS实例提供了秘密键。
- `session_token`： 选修的。为AWS实例提供了一套临时凭据。

这`parameters` 使用时需要以下选项`aws_sigv4` 验证：

- `region`：AWS实例所在的AWS区域。
- `service_name`：连接器的AWS服务的名称。

## cohere连接器

以下示例请求创建一个独立的cohere连接器：

```json
POST /_plugins/_ml/connectors/_create
{
  "name": "<YOUR CONNECTOR NAME>",
  "description": "<YOUR CONNECTOR DESCRIPTION>",
  "version": "<YOUR CONNECTOR VERSION>",
  "protocol": "http",
  "credential": {
    "cohere_key": "<YOUR Cohere API KEY HERE>"
  },
  "parameters": {
    "model": "embed-english-v2.0",
    "truncate": "END"
  },
  "actions": [
    {
      "action_type": "predict",
      "method": "POST",
      "url": "https://api.cohere.ai/v1/embed",
      "headers": {
        "Authorization": "Bearer ${credential.cohere_key}"
      },
      "request_body": "{ \"texts\": ${parameters.texts}, \"truncate\": \"${parameters.truncate}\", \"model\": \"${parameters.model}\" }", 
      "pre_process_function": "connector.pre_process.cohere.embedding",
      "post_process_function": "connector.post_process.cohere.embedding"
    }
  ]
}
```
{% include copy-curl.html %}

## 亚马逊基岩连接器

以下示例请求创建独立的Amazon基岩连接器：

```json
POST /_plugins/_ml/connectors/_create
{
  "name": "Amazon Bedrock Connector: embedding",
  "description": "The connector to the Bedrock Titan embedding model",
  "version": 1,
  "protocol": "aws_sigv4",
  "parameters": {
    "region": "<YOUR AWS REGION>",
    "service_name": "bedrock"
  },
  "credential": {
    "access_key": "<YOUR AWS ACCESS KEY>",
    "secret_key": "<YOUR AWS SECRET KEY>",
    "session_token": "<YOUR AWS SECURITY TOKEN>"
  },
  "actions": [
    {
      "action_type": "predict",
      "method": "POST",
      "url": "https://bedrock-runtime.us-east-1.amazonaws.com/model/amazon.titan-embed-text-v1/invoke",
      "headers": {
        "content-type": "application/json",
        "x-amz-content-sha256": "required"
      },
      "request_body": "{ \"inputText\": \"${parameters.inputText}\" }",
      "pre_process_function": "\n    StringBuilder builder = new StringBuilder();\n    builder.append(\"\\\"\");\n    String first = params.text_docs[0];\n    builder.append(first);\n    builder.append(\"\\\"\");\n    def parameters = \"{\" +\"\\\"inputText\\\":\" + builder + \"}\";\n    return  \"{\" +\"\\\"parameters\\\":\" + parameters + \"}\";",
      "post_process_function": "\n      def name = \"sentence_embedding\";\n      def dataType = \"FLOAT32\";\n      if (params.embedding == null || params.embedding.length == 0) {\n        return params.message;\n      }\n      def shape = [params.embedding.length];\n      def json = \"{\" +\n                 \"\\\"name\\\":\\\"\" + name + \"\\\",\" +\n                 \"\\\"data_type\\\":\\\"\" + dataType + \"\\\",\" +\n                 \"\\\"shape\\\":\" + shape + \",\" +\n                 \"\\\"data\\\":\" + params.embedding +\n                 \"}\";\n      return json;\n    "
    }
  ]
}
```
{% include copy-curl.html %}

## 下一步

- 要了解有关在OpenSearch中使用模型的更多信息，请参见[在OpenSearch中使用ML模型]({{site.url}}{{site.baseurl}}/ml-commons-plugin/ml-framework/)。
- 要了解有关模型访问控制和模型组的更多信息，请参见[模型访问控制]({{site.url}}{{site.baseurl}}/ml-commons-plugin/model-access-control/)。


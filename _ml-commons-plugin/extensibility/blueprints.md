---
layout: default
title: 连接器蓝图
has_children: false
nav_order: 65
parent: 连接到远程模型
---

# 连接器蓝图
**引入2.9**
{: .label .label-purple }

所有连接器均由机器学习（ML）开发人员创建的JSON蓝图组成。蓝图允许管理员和数据科学家在OpenSearch和AI服务或模型之间建立联系-服务技术。

以下示例显示了亚马逊萨吉式制造商连接器的蓝图：

```json
POST /_plugins/_ml/connectors/_create
{
  "name": "<YOUR CONNECTOR NAME>",
  "description": "<YOUR CONNECTOR DESCRIPTION>",
  "version": "<YOUR CONNECTOR VERSION>",
  "protocol": "aws_sigv4",
  "credential": {
    "access_key": "<YOUR AWS ACCESS KEY>",
    "secret_key": "<YOUR AWS SECRET KEY>",
    "session_token": "<YOUR AWS SECURITY TOKEN>"
  },
  "parameters": {
    "region": "<YOUR AWS REGION>",
    "service_name": "sagemaker"
  },
  "actions": [
    {
      "action_type": "predict",
      "method": "POST",
      "headers": {
        "content-type": "application/json"
      },
      "url": "<YOUR SAGEMAKER MODEL ENDPOINT URL>",
      "request_body": "<YOUR REQUEST BODY. Example: ${parameters.inputs}>"
    }
  ]
}
```
{% include copy-curl.html %}

## 示例蓝图

您可以在每个连接器中找到蓝图[ML Commons存储库](https://github.com/opensearch-project/ml-commons/tree/2.x/docs/remote_inference_blueprints)。

## 配置选项

以下配置选项是**必需的** 为了构建连接器蓝图。这些设置可用于独立和内部连接器。

| 场地| 数据类型| 描述|
| ：---  | :--- | :--- |
| `name` | 细绳| 连接器的名称。|
| `description` | 细绳| 连接器的描述。|
| `version` | 整数| 连接器的版本。|
| `protocol` | 细绳| 连接的协议。对于AWS服务，例如Amazon Sagemaker和Amazon Bedrock，请使用`aws_sigv4`。对于所有其他服务，请使用`http`。|
| `parameters` | JSON对象| 默认连接器参数，包括`endpoint` 和`model`。该字段中指示的任何参数都可以被预测请求中指定的参数覆盖。|
| `credential` | JSON对象| 定义为连接到所选端点所需的任何凭证变量。ML Commons使用**AES/GCM/nopadding** 对称加密以加密您的凭据。当与群集的连接首次启动时，OpenSearch创建了一个随机32-在OpenSearch的系统索引中持续存在的字节加密密钥。因此，您无需手动设置加密密钥。|
| `actions` | Json Array| 定义连接器内可以进行哪些操作。如果您是建立连接的管理员，请添加[蓝图]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/blueprints/) 为您所需的连接。|
| `backend_roles` | Json Array| OpenSearch后端角色的列表。有关设置后端角色的更多信息，请参见[将后端角色分配给用户]({{site.url}}{{site.baseurl}}/ml-commons-plugin/model-access-control#assigning-backend-roles-to-users)。|
| `access_mode` | 细绳| 设置模型的访问模式`public`，`restricted`， 或者`private`。默认为`private`。有关有关的更多信息`access_mode`， 看[模型组]({{site.url}}{{site.baseurl}}/ml-commons-plugin/model-access-control#model-groups)。|
| `add_all_backend_roles` | 布尔| 设置为`true`，添加全部`backend_roles` 到“访问列表”，只有具有管理权限的用户才能调整。设置为`false`，非-管理员可以添加`backend_roles`。|

这`action` 参数支持以下选项。

| 场地| 数据类型| 描述|
| ：---  | :--- | :--- |
| `action_type` | 细绳| 必需的。设置ML CONSONS API操作以在连接时使用。从OpenSearch 2.9开始，仅`predict` 得到支持。|
| `method` | 细绳| 必需的。定义API调用的HTTP方法。支持`POST` 和`GET`。|
| `url` | 细绳| 必需的。设置操作发生的连接端点。这必须匹配当时使用的连接的正则表达式[添加可信赖的端点]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/index#adding-trusted-endpoints)。|
| `headers` | JSON对象| 设置请求或响应主体内使用的标头。默认为`ContentType: application/json`。如果你的第三-政党ML工具需要访问控制，定义所需的`credential` 参数`headers` 范围。|
| `request_body` | 细绳| 必需的。设置操作请求主体内包含的参数。参数必须包括`\"inputText\`，指定连接器的用户应如何构建请求有效负载`action_type`。|
| `pre_process_function` | 细绳|  选修的。一个建筑-在或定制的无痛脚本中，用于预处理数据。OpenSearch提供以下构建的-在可以直接调用的预处理功能中：<br>- `connector.pre_process.cohere.embedding` 为了[共同](https://cohere.com/) 嵌入模型<br>- `connector.pre_process.openai.embedding` 为了[Openai](https://openai.com/) 嵌入模型<br>- `connector.pre_process.default.embedding`，您可以在神经搜索请求中使用它来预处理文档，以便它们以ML Commons可以使用默认的预处理程序进行处理（OpenSearch 2.11或更高版本）。有关更多信息，请参阅[建造-在功能中](#built-in-pre--and-post-processing-functions)。|
| `post_process_function` | 细绳| 选修的。一个建筑-在或定制的无痛脚本中用于发布-处理模型输出数据。OpenSearch提供以下构建的-在帖子中-您可以直接调用的过程功能：<br>- `connector.pre_process.cohere.embedding` 为了[共同文字嵌入模型](https://docs.cohere.com/reference/embed)<br>- `connector.pre_process.openai.embedding` 为了[Openai文本嵌入模型](https://platform.openai.com/docs/api-reference/embeddings) <br>- `connector.post_process.default.embedding`，您可以用来发布-模型响应中的处理文档，使它们处于神经搜索期望的格式（OpenSearch 2.11或更高版本）。有关更多信息，请参阅[建造-在功能中](#built-in-pre--and-post-processing-functions)。|

## 建造-在pre- 和张贴-处理功能

打电话给内置-在pre- 和张贴-处理功能，而不是在连接到以下文本嵌入模型或您自己的文本嵌入模型时，而不是编写自定义的无痛脚本（例如，亚马逊萨吉马克）：

- [OpenAI远程型号](https://platform.openai.com/docs/api-reference/embeddings)
- [共同远程模型](https://docs.cohere.com/reference/embed)

OpenSearch提供以下PRE- 和张贴-处理功能：

- Openai：`connector.pre_process.openai.embedding` 和`connector.post_process.openai.embedding`
- cohere：`connector.pre_process.cohere.embedding` 和`connector.post_process.cohere.embedding`
- [默认](#default-pre--and-post-processing-functions) （用于神经搜索）：`connector.pre_process.default.embedding` 和`connector.post_process.default.embedding`

### 默认pre- 和张贴-处理功能

当您使用矢量搜索时[神经搜索]({{site.url}}{{site.baseurl}}/search-plugins/neural-search/)，首先将神经搜索请求路由到ML CONSON，然后将其连接到模型。如果模型是[审计的模型由OpenSearch提供]({{site.url}}{{site.baseurl}}/ml-commons-plugin/pretrained-models/)，它可以解析ML Commons请求，并以ML Commons期望的格式返回响应。但是，对于远程模型，预期格式可能与ML Commons格式不同。默认pre- 和张贴-处理功能转化为模型期望的格式和神经搜索期望的格式。

#### 示例请求

以下示例请求创建一个sagemaker文本嵌入连接器并调用默认帖子-处理功能：

```json
POST /_plugins/_ml/connectors/_create
{
  "name": "Sagemaker text embedding connector",
  "description": "The connector to Sagemaker",
  "version": 1,
  "protocol": "aws_sigv4",
  "credential": {
    "access_key": "<YOUR SAGEMAKER ACCESS KEY>",
    "secret_key": "<YOUR SAGEMAKER SECRET KEY>",
    "session_token": "<YOUR AWS SECURITY TOKEN>"
  },
  "parameters": {
    "region": "ap-northeast-1",
    "service_name": "sagemaker"
  },
  "actions": [
    {
      "action_type": "predict",
      "method": "POST",
      "url": "sagemaker.ap-northeast-1.amazonaws.com/endpoints/",
      "headers": {
        "content-type": "application/json"
      },
      "post_process_function": "connector.post_process.default.embedding",
      "request_body": "${parameters.input}"
    }
  ]
}
```
{% include copy-curl.html %}

这`request_body` 模板必须是`${parameters.input}`。
{: .important}

### 预处理功能

这`connector.pre_process.default.embedding` 默认的预处理函数解析了神经搜索请求，并将其转换为模型期望为输入的格式。

ML共同体[预测API]({{site.url}}{{site.baseurl}}/ml-commons-plugin/api/train-predict/predict/) 提供以下格式的参数：

```json
{
  "parameters": {
    "input": ["hello", "world"]
  }
}
```

默认的预处理函数发送`input` 模型的字段内容。因此，模型输入格式必须是字符串列表，例如：

```json
["hello", "world"]
```

### 邮政-处理功能

这`connector.post_process.default.embedding` 默认帖子-处理函数解析模型响应，并将其转换为神经搜索所期望的输入的格式。

远程文本嵌入模型输出必须是两个-维浮点数组，每个元素代表输入列表中字符串的嵌入。例如，以下两个-尺寸数组对应于列表的嵌入`["hello", "world"]`：

```json
[
  [
    -0.048237994,
    -0.07612697,
    ...
  ],
  [
    0.32621247,
    0.02328475,
    ...
  ]
]
```

## 自定义pre- 和张贴-处理功能

你可以写自己的前- 和张贴-处理专门针对模型格式的功能。例如，以下亚马逊基石连接器定义包含自定义PRE- 和张贴-亚马逊基岩泰坦嵌入模型的处理功能：

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

有关创建各种连接器的示例，请参见[连接器]({{site.url}}{{site.baseurl}}/ml-commons-plugin/extensibility/connectors/)。

